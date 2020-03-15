# Reference
Splunk Enterprise system stack with S3 input/output. Designed for RHEL / CentOS 7 on EC2 instances. Includes master, search head, indexer cluster, and heavy forwarder(s). Features optional LDAP authentication and heavy forwarder proxy (for S3).

# Variables
```
# installation sources (s3)
installation_bucket, e.g.: mypackages
splunk_rpm_path, e.g.: splunk/splunk.rpm
splunk_aws_addon_path, e.g.: splunk/awsaddon.tgz
 
# required for some plays
web_key, e.g.: /etc/pki/tls/private/splunk.key
web_certificate, e.g.: /etc/pki/tls/private/splunk.pem
master_address, e.g.: splunk.chadg.net
splunk_secret, e.g.: somefancystring
aws_region, e.g. us-east-2
input_bucket, e.g. chadsbucket
input_directory, e.g. chadsdir
instance_profile, e.g. chad_splunk
output_bucket, e.g. splunksbucket

# optional for heavy forwarder access to s3
amazonaws.com due to lack of endpoint specification. (heavy_forwarder)
addon_proxy, e.g.: True
proxy_user, e.g.: chadsproxy
proxy_pass, e.g.: chadssecret
proxy_url, e.g.: proxy.chadg.net
proxy_port, e.g.: 499

# optional for ldap authentication (web and/or master)
ldap_auth, e.g.: True
ldap_host, e.g.: ldap.chadg.net
bind_dn, e.g.: uid=chadbind,cn=users,cn=accounts,dc=chadg,dc=net
bind_dn_password, e.g.: bindssecret
user_base_dn, e.g.: cn=users,dc=chadg,dc=net
admin_users, e.g: chad;user2
```

# Additional
- Place license file as conf/master/splunk.license
- Place web private key and certificate on master/web before running playbook.
- Use private endpoints for indexer communication to S3.
- Use proxy (or public access) for heavy forwarder communication to S3. The included proxy option uses a small script to add the proxy credentials to heavy forwarder(s).
- The indexer and heavy forwarder require an instance profile with S3 permissions to read/write input and output buckets/directories. It only needs to be defined in heavy forwarder plays.

# Deployment
master
```
# locally without ldap
ansible-playbook master.yml --extra-vars "target=localhost \
installation_bucket=mybucket \
splunk_rpm_path=splunk/splunk.rpm \
master_address=splunk.chadg.net \
splunk_secret=somefancystring \
aws_region=us_east_2 \
input_bucket=chads_bucket \
input_directory=chads_dir \
ldap_auth=False \
output_bucket=splunksbucket \
web_key=/etc/pki/tls/private/splunk.key \
web_certificate=/etc/pki/tls/private/splunk.pem"
```
web
```
# locally with ldap
ansible-playbook master.yml --extra-vars "target=localhost \
installation_bucket=mybucket \
splunk_rpm_path=splunk/splunk.rpm \
master_address=splunk.chadg.net \
splunk_secret=somefancystring \
aws_region=us_east_2 \
ldap_auth=True \
ldap_host=ldap.chadg.net \
bind_dn=uid=chadbind,cn=users,cn=accounts,dc=chadg,dc=net \
bind_dn_password=bindssecret \
user_base_dn=cn=users,dc=chadg,dc=net \
admin_users=chad \
web_key=/etc/pki/tls/private/splunk.key \
web_certificate=/etc/pki/tls/private/splunk.pem"
```
indexer(s)
```
# locally
ansible-playbook indexer.yml --extra-vars "target=localhost \
installation_bucket=mybucket \
splunk_rpm_path=splunk/splunk.rpm \
master_address=splunk.chadg.net \
splunk_secret=somefancystring \
aws_region=us_east_2"
```
heavy_forwarder(s)
Scale large bucket ingestion with additional forwarders, e.g.:
```
# locally, first heavy forwarder ingests s3://chadsbucket1/data1
ansible-playbook indexer.yml --extra-vars "target=localhost \
installation_bucket=mybucket \
splunk_rpm_path=splunk/splunk.rpm \
splunk_aws_addon_path=splunk/awsaddon.tgz \
master_address=splunk.chadg.net \
splunk_secret=somefancystring \
aws_region=us_east_2 \
instance_profile=chad_splunk \
input_bucket=chadsbucket1 \
input_directory=data1 \
addon_proxy=True \
proxy_user=chadsproxy \
proxy_pass=chadssecret \
proxy_url=proxy.chadg.net \
proxy_port=499"

# locally, second heavy forwader ingests s3://chadsbucket2/data2
ansible-playbook indexer.yml --extra-vars "target=localhost \
installation_bucket=mybucket \
splunk_rpm_path=splunk/splunk.rpm \
splunk_aws_addon_path=splunk/awsaddon.tgz \
master_address=splunk.chadg.net \
splunk_secret=somefancystring \
aws_region=us_east_2 \
instance_profile=chad_splunk \
input_bucket=chadsbucket2 \
input_directory=data2 \
addon_proxy=True \
proxy_user=chadsproxy \
proxy_pass=chadssecret \
proxy_url=proxy.chadg.net \
proxy_port=499"
```

# Todo
Auto-generate self-signed SSL when certificates not provided.
