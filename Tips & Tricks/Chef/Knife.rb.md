
```
ilog_level               :info
log_location             STDOUT
node_name                'euthuppan'
client_key               '/Users/euthuppan/.chef/qa.pem'
validation_client_name   'salesforce-validator'
validation_key           '/Users/euthuppan/.chef/salesforce-validator.pem'
chef_server_url          'https://chef.qa.local:443/organizations/salesforce'
syntax_check_cache_path  '/Users/euthuppan/.chef/syntax_check_cache'
cookbook_path [ '~/src/chef-cookbooks']
knife[:ssh_user] = 'etstaff'
knife[:ssh_password] = "Overrule-Snooper-Mayflower-Sandbox-Petal-Gills"

knife[:editor] = '/usr/bin/vim'
knife[:concurrency] = 10
knife[:secret_file] = '/Users/euthuppan/src/chef-cookbooks/databag-0.2.0/files/default/qa_encrypted_data_bag_secret'
ssh_timeout 5
```

