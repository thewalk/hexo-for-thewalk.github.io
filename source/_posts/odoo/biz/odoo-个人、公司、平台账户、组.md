---
title: '[odoo]个人、公司、平台账户、组'
date: 2020-03-11 15:22:00
tags: 
- odoo
- biz
category:
- odoo
---

# 个人、公司与平台账户

odoo中实际的商业个体（个人或公司）与平台账户是拆分开的，根据对数据表的拆分，可以看到：

- 所有的商业个体（个人或者公司）都记录在**res_partner**表中，如果是公司，还有**res_company**作为额外的数据表使用

- 平台账户具备单独的表，只有记录在该表中的账号才可以登陆，使用的数据表为**res_users**

- 平台账户可以与具体的某商业个体（个人或者公司）进行关联，也可以不关联。如果进行了关联，则可以理解为该账户由某个商业个体使用。

按照上述理解能够对odoo的用户、公司进行良好的管理，能够有效的区分平台账户与商业个体各自的信息。

# 组

商业个体（个人、公司）都是针对业务的，对于平台管理并没有什么作用，平台最终管理的对象还是平台账户，而组则是建立在平台账户之上的。

odoo中组是被设计成可继承的，暂时还不了解odoo对于嵌套组中权限冲突是如何解决的（根据经验，不推荐使用嵌套组）