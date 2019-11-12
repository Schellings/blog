---
title: MySQL插入解决主键冲突，预防sql执行报错
date: 2018-02-18 16:56:24
tags: MySQL
---

两种方案：

```yaml
INSERT INTO `idm_app`
SELECT 'iam', 'IAM应用', 'default', 'default', 1, NULL, NULL, NULL, NULL, '2019-07-17 16:36:31', NULL, '2019-07-17 16:36:31', NULL, '{}',NULL
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `idm_app` WHERE id = 'iam')


INSERT INTO `idm_app` VALUES ('iam2', 'IAM应用2', 'default', 'default', 1, NULL, NULL, NULL, NULL, '2019-07-17 16:36:31', NULL, '2019-07-17 16:36:31', NULL, '{}',NULL) ON DUPLICATE KEY UPDATE `updated_at` = NOW();
```