# Table of Contents #


# Quick Start Instructions #

  1. Download the latest Capirca bundle:
    * http://code.google.com/p/capirca/downloads/list
  1. Extract the files
```
tar -xvzf capirca-rXXX.tgz
```
  1. Move into the extracted directory
```
cd ./capirca
```
  1. Generate filters for the targets in the provided sample policy
```
./aclgen.py   (alternately 'python ./aclgen.py')
```
  1. Examine the resulting output filters
```
ls ./filters
vi ./filters/sample.*
```

# Creating Your Own Filters #

  1. Add your Networks & Hosts definitions
```
vi ./def/NETWORK.net
```
    * Note: you can also create your own [CUSTOM](CUSTOM.md).net files which will be automatically included.  This can be used to group or separate particular definitions into distinct files.
  1. Add any necessary service definitions
```
vi ./def/SERVICES.svc
```
    * Note: you can also create your own [CUSTOM](CUSTOM.md).svc files which will be automatically included.
  1. Define a new security policy
```
vi ./policies/my-custom.pol
```
  1. Create the filter header and specify target platforms
    * http://code.google.com/p/capirca/wiki/PolicyFormat#Terms_Section
```
header {
  comment:: "Speedway generates iptables filter suitable for passing"""
  comment:: "to iptables-restore."""
}
```
  1. Define the policy rules / terms
    * http://code.google.com/p/capirca/wiki/PolicyFormat#Terms_Section
```
term allow-inbound-ssh {
  destination-address: MY_SERVERS
  protocol:: tcp
  destination-port:: SSH
  action:: accept
}
```
  1. Generate your filter
```
./aclgen.py
```
  1. Review and Check the generated filter
```
vi ./filters/my-custom.ipt
```

# Help #

Discussion Forum: http://groups.google.com/group/capirca-dev

Group email: capirca-dev@googlegroups.com