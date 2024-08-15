# SSTI(Server-Side Template Injection)

## Python

### Flask-Jinja2

未过滤参数，python使用Jinja2等库，调用Template渲染页面，未校验用户输入，接受用户参数作为渲染输入。可通过`{{(3*3)}}`等操作尝试是否执行计算。

python Flask-jinja2 关键字绕过：[以 Bypass 为中心谭谈 Flask-jinja2 SSTI 的利用 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/9584?time__1311=n4%2BxnD0DuA5CwhDBqootGQ3G%3DDCDcG6ZEnxmwpD)

[Marven11/Fenjing: 专为CTF设计的Jinja2 SSTI全自动绕WAF脚本 | A Jinja2 SSTI cracker for bypassing WAF, designed for CTF (github.com)](https://github.com/Marven11/Fenjing)

