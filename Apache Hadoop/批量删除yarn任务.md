# 批量删除yarn任务

```bash
for i in  `yarn application  -list | grep -w  ACCEPTED | awk '{print $1}' | grep application_`; do yarn  application -kill $i; done
```
