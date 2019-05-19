This repo contains instructions and an Ansible playbook to configure an Asterisk server. It was used to test Nexmo's SIP Connect functionality.

**THIS REPO MAY CONFIGURE ASTERISK IN A NON-SECURE FASHION. USE AT YOUR OWN RISK**

For those of you testing, make sure you delete your machine once you're done by running

```
doctl compute droplet delete asterisk --context nexmo
```

# Configuration

Before we get started let's specify how we'd like our Asterisk server configured.

## Authentication

There are two sets of authentication credentials involved when setting up Asterisk; your Asterisk user password and your Nexmo credentials (used for the SIP connection).

Edit `config.yml` and change the value of `asterisk_password`. Choose a strong password, as SIP scanners are constantly looking for boxes to make calls through and it could cost a lot of $$$ if they hack in.

You'll also need to edit `nexmo_api_key` and `nexmo_api_secret` so that Asterisk can connect to the Nexmo platform.

## Configuring your extensions

Edit `config.yml` and define your required extensions. Replace the numbers in the examples with the LVN linked to your application.

If you need to pass any additional headers, you can define them as key:value pairs in this config file

```yaml
extensions:
 - extension: 69100
   endpoint: nexmo/14155550100
   headers:
       X-Badgers: Furry
 - extension: 69200
   endpoint: nexmo/442079460000
```

# Creating a DigitalOcean droplet

We use a DigitalOcean droplet for testing as there are no firewalls involved by default. This prevents any NAT issues for Asterisk.

This guide uses the Digital Ocean CLI. You can install it with `brew install doctl`. For non-MacOS machines see [installing doctl](https://github.com/digitalocean/doctl#installing-doctl).

To use the API you'll need to create an API auth token on https://cloud.digitalocean.com/account/api

Once you've created one, create a new `nexmo` context for `doctl`. This will allow you to have both Nexmo and personal doctl sessions active.

```
doctl auth init nexmo
```

To access any machine you create you'll need to register some SSH keys with DigitalOcean. Upload your public key at  https://cloud.digitalocean.com/account/security and make a note of the SSH key fingerprint, we'll need it for the next step.

Now we can create a droplet and install Python (which we need to run our Ansible playbook later). Make sure to replace `SSH KEY FINGERPRINT` with the fingerprint from the last step.

```
doctl --context nexmo compute droplet create asterisk --image ubuntu-18-10-x64 --region lon1 --size 1gb --ssh-keys "<SSH KEY FINGERPRINT>"
doctl --context nexmo compute ssh asterisk --ssh-command "apt-get install python -y"
```

Once the machine is created we need to get the machine's IP address to connect to it. Run the following and make a note of the `asterisk` machine's IP address:

```
doctl --context nexmo compute droplet list
```

# Running Ansible

Now that we have a machine we can run Ansible to install and configure Asterisk (you'll need to [install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) if you don't already use it).

As we're using SSH keys, authentication should be taken care of. To install Asterisk run the following in the same directory as `playbook.yml`, replacing `MACHINE_IP` with your DigitalOcean machine IP (careful, the comma here is important so make sure to include it!)

Fingers crossed everything works, and in a few minutes you should have a configured Asterisk server.

```
ansible-playbook -i '<MACHINE_IP>,' playbook.yml
```

# Connecting using Telephone (MacOS)

[Telephone](https://itunes.apple.com/us/app/telephone/id406825478?mt=12) is a free MacOS SIP client that you can use to connect to your Asterisk server.

Once you've installed Telephone, add a new account with the following details

* `Full Name`: <Your Name>
* `Domain`: <MACHINE_IP> (from DigitalOcean)
* `User Name`: 6001
* `Password`: <ASTERISK_PASSWORD> (that you set in `config.yml`)

Once this is added you should see a window that says `Available` at the top. In this window you can dial any extensions you configured (e.g. `69100`) and it will forward you on to your Nexmo LVN.

If you're having any issues connecting to your Nexmo LVN, call `68600` to check that your Asterisk server is working correctly. This extension plays a debug message to you.
