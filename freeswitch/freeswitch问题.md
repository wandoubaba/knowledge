# freeswitch问题（和解决）

---

[toc]

## 网内通话，在conf/dialplan/default.xml中设置的拨号计划不生效，只走public.xml

在`conf/sip_profiles/internal.xml`文件中：

把`<param name="context" value="public"/>`改成`<param name="context" value="default"/>`

> 它默认是public
