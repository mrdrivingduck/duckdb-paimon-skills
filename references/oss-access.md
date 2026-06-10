# OSS Remote Warehouse Access

## Credential File

Ask the user to save their OSS credentials in a file (any path), using this format:

```
KEY_ID=your-access-key-id
SECRET=your-access-key-secret
ENDPOINT=oss-cn-hangzhou.aliyuncs.com
```

Read the file, parse the three values, and construct the `CREATE SECRET` SQL:

```sql
CREATE SECRET my_oss (
    TYPE paimon,
    KEY_ID 'your-access-key-id',
    SECRET 'your-access-key-secret',
    ENDPOINT 'oss-cn-hangzhou.aliyuncs.com'
);
```

Common OSS endpoints:

| Region | Endpoint |
|--------|----------|
| Hangzhou | `oss-cn-hangzhou.aliyuncs.com` |
| Shanghai | `oss-cn-shanghai.aliyuncs.com` |
| Beijing | `oss-cn-beijing.aliyuncs.com` |
| Shenzhen | `oss-cn-shenzhen.aliyuncs.com` |
| Zhangjiakou | `oss-cn-zhangjiakou.aliyuncs.com` |

## Attach Remote Warehouse

```sql
ATTACH 'oss://your-bucket/warehouse' AS my_catalog (TYPE paimon);
```

Then use standard SQL to browse and query:

```sql
SHOW ALL TABLES;
SELECT * FROM my_catalog.db_name.table_name LIMIT 10;
```

## Troubleshooting

### Access Denied

- Verify AccessKey ID and Secret are correct
- Ensure the AccessKey has `oss:GetObject` and `oss:ListObjects` permissions on the bucket
- Check the endpoint matches the bucket's region

### Network Timeout

- Verify OSS endpoint is reachable from your environment
- Use internal endpoints (`oss-cn-hangzhou-internal.aliyuncs.com`) when running inside Alibaba Cloud VPC

### Security Notes

- Never hardcode credentials in scripts or commit them to version control
- Use RAM role-based temporary credentials in production environments
- DuckDB Secret Manager redacts key values in logs automatically
