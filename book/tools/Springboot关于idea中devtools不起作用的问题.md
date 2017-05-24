关于idea中devtools不起作用的问题

相信大部分使用Intellij的同学都会遇到这个问题，即使项目使用了spring-boot-devtools，修改了类或者html、js等，idea还是不会自动重启，非要手动去make一下或者重启，

就更没有使用热部署一样。出现这种情况，并不是你的配置问题，相信自己，热部署那几个设置很简单，其根本原因是因为Intellij IEDA和Eclipse不同，Eclipse设置了自动编译之

后，修改类它会自动编译，而IDEA在非RUN或DEBUG情况下才会自动编译（前提是你已经设置了Auto-Compile）。

下面看步骤：

1. 设置自动编译：
   setting->compiler下将build project automa...勾选
2. Shift+Ctrl+Alt+/，选择Registry,
   将compiler.automake.allow.when.app.running和actionSystem.assertFocusAccessFromEdt勾选.








