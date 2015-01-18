Advertise/monitor Redis clusters with Consul
============================================

This is a "joiner" BOSH release to upgrade a [redis-boshrelease](https://github.com/cloudfoundry-community/redis-boshrelease) deployment to include consul agents on all machines and monitoring/advertising of redis processes.

This uses the [consul-boshrelease](https://github.com/cloudfoundry-community/consul-boshrelease), and assumes Redis is being deployed using this [redis-boshrelease](https://github.com/cloudfoundry-community/redis-boshrelease).

![consul-ui](http://cl.ly/image/2O1G361Z3M0E/redis_cluster_in_consul_ui.png)

Usage
-----

Instead of deploying only the `redis` job template from [redis-boshrelease](https://github.com/cloudfoundry-community/redis-boshrelease), the pattern to use is to combine three job templates from three BOSH releases.

-	`redis` from redis-boshrelease
-	`redis-consul` from consul-redis-boshrelease
-	`consul` from consul-boshrelease

This pattern of a "join release" between `redis` and `consul` means that these two BOSH releases can operate independently of each other, and the `redis-consul` release "joins" them together. Hence its called a "joiner" release.

Deployment
----------

You would use the spiff templates from this project, rather than the `redis-boshrelease`.

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/redis-consul-boshrelease.git
cd redis-consul-boshrelease
bosh upload release releases/redis-consul-2.yml
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a cluster:

```
templates/make_manifest warden
bosh -n deploy
```

For AWS EC2, create a single VM:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

```yaml
---
networks:
  - name: redis-consul1
    type: dynamic
    cloud_properties:
      security_groups:
        - redis-consul
```

Where `- redis-consul` means you wish to use an existing security group called `redis-consul`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```
