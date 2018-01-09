# Terraform on OpenStack

OpenStack could offer infrastructure for us. If we apply for a group of infrastructure, an agent talking to OpenStack is needed. That is [`Terraform`](https://www.terraform.io/). It also could works with other IaaS.

We gave Terraform a infrastructure blueprint in our mind, it will performs as you wish. Then change the blueprint, Terraform will do his best to complete blueprint base exist infrastructure.

## Terraform directory

This dirctory is a workspace. The blueprint file(`.tf`) and intermediate state file stored here. If you want maintain a group infrastructure for long time, you need save the whole directory. The absolute path of this directory is also changeless.

## Terraform file

Terraform file(`.tf`) store the information of who you are and what you want.

### Who You Are

Define `provider` to show who you are, just like below.

```shell
# with password
provider "openstack" {
  user_name   = "user_name"
  password    = "password"
  auth_url    = "http://1.1.1.1:5000/v3"
  region      = "RegionOne"
  domain_id   = "default"
  tenant_id = "aaaaaaaaaa"
}

# with token
provider "openstack" {
  token       = "aaaaaaa"
  auth_url    = "http://1.1.1.1:5000/v3"
  region      = "RegionOne"
}
```

Maybe we don't need option `domain_id` in different `OpenStack` environment. If there are errors we can not figure out, we could remove `region`, `domain_id` to have a try.

More detail in [Documents](https://www.terraform.io/docs/providers/openstack/)

### What You Want

Define a network, subnet, router, sec group.
Create instances on this network.

More detail on [Documents](https://www.terraform.io/docs/providers/openstack/) and a [blog](https://www.matt-j.co.uk/2015/03/27/openstack-infrastructure-automation-with-terraform-part-2/)

```shell
#
# Create a security group
#
 
resource "openstack_compute_secgroup_v2" "tf_sec_1" {
    region = ""
    name = "sec-name"
    description = "Security Group For "
    rule {
        from_port = 22
        to_port = 22
        ip_protocol = "tcp"
        cidr = "0.0.0.0/0"
    }
    rule {
        from_port = -1
        to_port = -1
        ip_protocol = "icmp"
        cidr = "0.0.0.0/0"
    }
    rule {
        from_port = 1
        to_port = 65535
        ip_protocol = "udp"
        self = true
    }
    rule {
        from_port = 1
        to_port = 65535
        ip_protocol = "tcp"
        self = true
    }
    rule {
        from_port = 443
        to_port = 443
        ip_protocol = "tcp"
        cidr = "0.0.0.0/0"
    }
}


#
# Create a Network
#
resource "openstack_networking_network_v2" "tf_network" {
    region = ""
    name = "network-name"
    admin_state_up = "true"
}

resource "openstack_networking_subnet_v2" "tf_net_sub1" {
    region = ""
    name = "subnet-name"
    network_id = "${openstack_networking_network_v2.tf_network.id}"
    cidr = "192.168.1.0/24"
    ip_version = 4
    dns_nameservers = ["114.114.114.114", "8.8.8.8"]
}

#
# Create a router for our network
#
resource "openstack_networking_router_v2" "tf_router1" {
    region = ""
    name = "router-name"
    admin_state_up = "true"
    external_gateway = "88787937-6924-4b36-b191-663e745a3666"
}

#
# Attach the Router to our Network via an Interface
#
resource "openstack_networking_router_interface_v2" "tf_rtr_if_1" {
    region = ""
    router_id = "${openstack_networking_router_v2.tf_router1.id}"
    subnet_id = "${openstack_networking_subnet_v2.tf_net_sub1.id}"
}

#
# Create some Openstack Floating IP's for our VM's
#
resource "openstack_compute_floatingip_v2" "fip_1" {
    region = ""
    pool = "FloatingFrom"
}

resource "openstack_compute_floatingip_associate_v2" "fip_1" {
  floating_ip = "${openstack_compute_floatingip_v2.fip_1.address}"
  instance_id = "${openstack_compute_instance_v2.master_instance.0.id}"
}

# variable "cluster_network_uuid" {
  # default = "${openstack_networking_network_v2.tf_network.id}"
# }


resource "openstack_compute_instance_v2" "master_instance" {
  count = 3
  name  = "${format("master-%02d", count.index)}"
  region    = ""
  image_id  = "vm-image-id"
  flavor_id = "flavor-id"
  security_groups = ["sec-name","default"]
  user_data = "#!/bin/sh\n touch file"
  config_drive = "true"

  network {
    uuid = "${openstack_networking_network_v2.tf_network.id}"
  }
}

resource "openstack_compute_instance_v2" "node_instance" {
  count = 0
  name = "${format("node-%02d", count.index)}"
  region    = ""
  image_id  = "vm-image-id"
  flavor_id = "flavor-id"
  security_groups = ["sec-name","default"]
  user_data = "#!/bin/sh\n touch file"
  config_drive = "true"

  network {
    uuid = "${openstack_networking_network_v2.tf_network.id}"
  }
}

resource "openstack_compute_instance_v2" "worker" {
  name      = "worker"
  region    = ""
  image_id  = "vm-image-id"
  flavor_name = "t2.4medium"
  security_groups = ["sec-name","default"]
  user_data = "#!/bin/sh\n touch file"
  config_drive = "true"

  network {
    uuid = "88787937-6924-4b36-b191-663e745a3666"
    name = "OutNetwork"
  }

  network {
    uuid = "${openstack_networking_network_v2.tf_network.id}"
  }
}


output "master_ips" {
  value = "${openstack_compute_instance_v2.master_instance.*.network.0.fixed_ip_v4}"
}
output "master_ids" {
  value = "${openstack_compute_instance_v2.master_instance.*.id}"
}
output "node_ips" {
  value = "${openstack_compute_instance_v2.node_instance.*.network.0.fixed_ip_v4}"
}
output "node_ids" {
  value = "${openstack_compute_instance_v2.node_instance.*.id}"
}
output "worker_ips_0" {
  value = "${openstack_compute_instance_v2.worker.network.0.fixed_ip_v4}"
}
output "worker_ips_1" {
  value = "${openstack_compute_instance_v2.worker.network.1.fixed_ip_v4}"
}
output "floating_ip" {
  value = "${openstack_compute_floatingip_associate_v2.fip_1.floating_ip}"
}
output "instance_network" {
  value = "${openstack_compute_instance_v2.master_instance.0.network.0.uuid}"
} 
output "subnet_id" {
  value = "${openstack_networking_subnet_v2.tf_net_sub1.id}"
}
output "router_id" {
  value = "${openstack_networking_router_v2.tf_router1.id}"
}
output "floating_ip_id" {
  value = "${openstack_compute_floatingip_v2.fip_1.id}"
}
output "sec_id" {
  value = "${openstack_compute_secgroup_v2.tf_sec_1.id}"
}
```

## Terraform CMD

After define `.tf` in workspace. You need setup provider plugin. Follow [Document](https://www.terraform.io/docs/plugins/provider.html) or copy plugin to `/bin`.

Run `terraform init`, `terraform plan` and `terraform apply`.

You will see output final.