Keep your Odoo data safe with this module. Take automated back-ups, remove them automatically. You can even specify how long local backups and external backups should be kept, automatically

# Installation
Before installing this module, you need to execute:

```pip3 install boto3```

# Configure

Go to Settings -> Technical -> System Parameters to add:

|             KEY                 |     VALUE       |
|---------------------------------|-----------------|
| backup_s3.aws_secret_access_key | your secret key |
| backup_s3.aws_access_key_id     |    your key     |

<!-- # Usage -->

*[Download](https://apps.odoo.com/apps/modules/12.0/auto_backup_aws_s3/)*