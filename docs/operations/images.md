# Images

The default images for Kops clusters are based on Debian 9 (Stretch). These images are built by the [kubernetes-sigs/image-builder](https://github.com/kubernetes-sigs/image-builder/tree/master/images/kube-deploy/imagebuilder) project for each Kubernetes release and are available for any other use under the `kope.io` account.

You can choose a different image for an instance group by editing it with `kops edit ig nodes`. You should see an `image` field in one of the following formats:

* `ami-abcdef` - specifies an AMI by id directly
* `<owner>/<name>` specifies an AMI by its owner's account ID  and name properties
* `<alias>/<name>` specifies an AMI by its owner's alias and name properties

Using the AMI id is precise, but ids vary by region. It is often more convenient to use the `<owner/alias>/<name>`, if equivalent images have been copied to various regions with the same name.

Kops supports owner aliases for the official accounts of supported distros:

* `kope.io` => `383156758163`
* `amazon` => `137112412989`
* `centos` => `679593333241`
* `debian9` => `379101102735`
* `debian10` => `136693071363`
* `flatcar` => `075585003325`
* `redhat` => `309956199498`
* `ubuntu` => `099720109477`

For example, to use Ubuntu 20.04, you could specify:

```yaml
image: ami-00579fbb15b954340
```

or

```yaml
image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20200423
```

or

```yaml
image: ubuntu/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20200423
```

You can find the name for an image using:

`aws ec2 describe-images --region us-east-1 --image-id ami-00579fbb15b954340`

## Kope.io

The default imagess from `kope.io` are based on Debian 9 (Stretch). These images include all the necessary files and packages to run Kubernetes, making node startup faster. Other than that, the changes to the official Debian images are (minimal)[https://github.com/kubernetes-sigs/image-builder/tree/master/images/kube-deploy/imagebuilder/templates].

The latest image names are kept in the [stable channel manifest](https://github.com/kubernetes/kops/blob/master/channels/stable), but all available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 383156758163 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=*-debian-stretch-*"
```

## Debian 9 (Stretch)

Debian 9 (Stretch) is fully supported and functional.

It is based on Kernel version **4.9** which has number of known bugs that affect it and may be noticed with larger clusters:

* https://github.com/kubernetes/kubernetes/issues/56903
* https://github.com/kubernetes/kubernetes/issues/67577

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 379101102735 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=debian-stretch-hvm-x86_64-gp2-*"
```

## Debian 10 (Buster)

Debian 10 support was introduced in **Kops 1.13** and is fully functional as long as `iptables` is in `legacy` mode.

It is based on Kernel version **4.19** which fixes some of the bugs present in Debian 9 and effects are less visible.

One notable change is the addition of `iptables` NFT, which is by default. This is not yet supported by most CNI plugins and also seems to be [slower](https://youtu.be/KHMnC3kj3Js?t=771) than the legacy version. It is recommended to switch to `iptables` legacy by using the following script in `additionalUserData` for each instance group:

```yaml
additionalUserData:
  - name: busterfix.sh
    type: text/x-shellscript
    content: |
      #!/bin/sh
      update-alternatives --set iptables /usr/sbin/iptables-legacy
      update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
      update-alternatives --set arptables /usr/sbin/arptables-legacy
      update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 136693071363 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=debian-10-amd64-*"
```

## Ubuntu 18.04 (Bionic)

Ubuntu 18.04 is fully supported and functional.

It is based on Kernel version **4.15** which has number of known bugs that affect it and may be noticed with larger clusters:

* https://github.com/kubernetes/kubernetes/issues/56903
* https://github.com/kubernetes/kubernetes/issues/67577

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 099720109477 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-*"
```

## Ubuntu 20.04 (Focal)

Ubuntu 20.04 support was introduced in **Kops 1.16** and is fully functional. 

It is based on Kernel version **5.4** which fixes all the know major Kernel bugs.

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 099720109477 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-*"
```

## CentOS 7

CentOS 7 is supported and functional. The minimum supported version is **7.4**.

It is based on Kernel version **3.10** which has a considerable number of known bugs that affect it and may be noticed in production clusters:

* https://github.com/kubernetes/kubernetes/issues/56903
* https://github.com/kubernetes/kubernetes/issues/67577

Before using CentOS images you must accept the agreement at https://aws.amazon.com/marketplace/pp?sku=aw0evgkw8e5c1q413zgy5pjce.

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 679593333241 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=product-code,Values=aw0evgkw8e5c1q413zgy5pjce" "Name=name,Values=CentOS*"
```

## RHEL 7

RHEL 7 is supported and functional. The minimum supported version is **7.4**.

It is based on Kernel version **3.10** which has a considerable number of known bugs that affect it and may be noticed in production clusters:

* https://github.com/kubernetes/kubernetes/issues/56903
* https://github.com/kubernetes/kubernetes/issues/67577

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 309956199498 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=RHEL-7.*x86_64*"
```

## RHEL 8

RHEL 8 support was introduced in **Kops 1.18** and is experimental. 

It is based on Kernel version **4.18** which fixes some of the bugs present in RHEL/CentOS 7 and effects are less visible.

One notable change is the addition of `iptables` NFT, which is the only iptables backend available. This is not yet supported by most CNI plugins and should be used with care.

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 309956199498 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=RHEL-8.*x86_64*"
```

## Amazon Linux 2

Amazon Linux 2 support is still experimental, but should work with **Kops 1.18** or newer. Please report any issues.

It is based on Kernel version **4.18** which fixes some of the bugs present in RHEL/CentOS 7 and effects are less visible.

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 137112412989 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=amzn2-ami-hvm-2*-x86_64-gp2"
```

## Flatcar

Flatcar is a friendly fork of CoreOS and as such, compatible with it.

Support was introduced in **Kops 1.15** and it is both supported and functional.

Available images can be listed using:

```bash
aws ec2 describe-images --region us-east-1 --output table \
  --owners 075585003325 \
  --query "sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId]" \
  --filters "Name=name,Values=Flatcar-stable-*-hvm"
```

## Debian 8 (Jessie)

Support for Debian 8 (Jessie) is deprecated and will be removed in future versions of Kops.

## Ubuntu 16.04 (Xenial)

Support for Ubuntu 16.04 (Xenial) is deprecated and will be removed in future versions of Kops.

## CoreOS

Support for CoreOS is deprecated and will be removed in future versions of Kops.

You should consider using Flatcar as a replacement.

