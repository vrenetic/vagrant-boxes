Vagrant boxes
=============
Generate [Vagrant](http://www.vagrantup.com/) boxes with [packer](http://www.packer.io/).

Boxes are built and released on [Vagrant Cloud](https://vagrantcloud.com/vrenetic) for `virtualbox` and `aws`.

**Debian 8 (jessie):**
- `debian-8-amd64-plain`: Minimalistic
- `debian-8-amd64-default`: With *git*, *rsync*, *ruby* and *puppet*
- `debian-8-amd64-cm`: With [CM framework](https://github.com/vrenetic/cm) dependencies.

**Ubuntu 16.04 (Xenial Xerus):**
- `ubuntu-1604-plain`: Minimalistic
- `ubuntu-1604-default`: With *git*, *rsync*, *ruby* and *puppet*

**Ubuntu 15.04 (Vivid Vervet):**
- `ubuntu-1504-plain`: Minimalistic
- `ubuntu-1504-default`: With *git*, *rsync*, *ruby* and *puppet*


Usage: Virtualbox
-----------------

Example `Vagrantfile`:
```ruby
Vagrant.configure('2') do |config|
  config.vm.box = 'vrenetic/debian-8-amd64-default'
end
```


Usage: AWS
----------
Based on [official Debian AMIs](https://wiki.debian.org/Cloud/AmazonEC2Image/Wheezy).
Using the *HVM EBS* flavor as [recommended](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/virtualization_types.html) by Amazon.
Available regions: `eu-west-1`, `us-east-1`.

Example `Vagrantfile` (using the [vagrant AWS provider plugin](https://github.com/mitchellh/vagrant-aws)):
```ruby
Vagrant.configure('2') do |config|
  config.vm.box = 'vrenetic/debian-8-amd64-default'

  config.vm.provider :aws do |aws, override|
    override.ssh.username = 'admin' # For ubuntu use 'ubuntu'
    override.ssh.private_key_path = '~/.ssh/<private-key>.pem'

    aws.region = 'eu-west-1'
    aws.instance_type = 'm4.large'
    aws.ami = '' # due to a bug, see https://github.com/mitchellh/vagrant-aws/issues/330
    aws.access_key_id = '<aws-access-key>'
    aws.secret_access_key = '<aws-secret-key>'
    aws.keypair_name = '<keypair-name>'
    aws.security_groups = '<security-group-id>'

    aws.block_device_mapping = [
      {
        'DeviceName' => '/dev/xvda',
        'VirtualName' => 'root',
        'Ebs.VolumeSize' => 100,
        'Ebs.DeleteOnTermination' => true,
        'Ebs.VolumeType' => 'io1',
        'Ebs.Iops' => 2000
      }
    ]
  end
end
```


Development (building and uploading)
------------------------------------
Install dependencies:
```
bundle install
```

Download required puppet modules using [librarian-puppet](http://librarian-puppet.com/):
```
cd puppet
bundle exec librarian-puppet update
```

Rake parameters:
- template: Name of the template to build (Default: all templates)
- builder: A list of builders to use (Default: all builders)
- aws_key_id: AWS key id
- aws_key_secret: AWS key secret
- vagrant_cloud_username: Vagrant Cloud username
- vagrant_cloud_access_token: Vagrant Cloud access token

```
bundle exec rake build    # Build all boxes
bundle exec rake spec     # Run serverspec tests (virtualbox build only!)
bundle exec rake release  # Release boxes to S3 and Vagrant Cloud
```
