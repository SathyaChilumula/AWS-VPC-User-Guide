# Security Groups for Your VPC<a name="VPC_SecurityGroups"></a>

A *security group* acts as a virtual firewall for your instance to control inbound and outbound traffic\. When you launch an instance in a VPC, you can assign up to five security groups to the instance\. Security groups act at the instance level, not the subnet level\. Therefore, each instance in a subnet in your VPC could be assigned to a different set of security groups\. If you don't specify a particular group at launch time, the instance is automatically assigned to the default security group for the VPC\. 

For each security group, you add *rules* that control the inbound traffic to instances, and a separate set of rules that control the outbound traffic\. This section describes the basic things you need to know about security groups for your VPC and their rules\.

You might set up network ACLs with rules similar to your security groups in order to add an additional layer of security to your VPC\. For more information about the differences between security groups and network ACLs, see [Comparison of Security Groups and Network ACLs](VPC_Security.md#VPC_Security_Comparison)\.

**Topics**
+ [Security Group Basics](#VPCSecurityGroups)
+ [Default Security Group for Your VPC](#DefaultSecurityGroup)
+ [Security Group Rules](#SecurityGroupRules)
+ [Differences Between Security Groups for EC2\-Classic and EC2\-VPC](#VPC_Security_Group_Differences)
+ [Working with Security Groups](#WorkingWithSecurityGroups)

## Security Group Basics<a name="VPCSecurityGroups"></a>

The following are the basic characteristics of security groups for your VPC:
+ You have limits on the number of security groups that you can create per VPC, the number of rules that you can add to each security group, and the number of security groups you can associate with a network interface\. For more information, see [Amazon VPC Limits](amazon-vpc-limits.md)\.
+ You can specify allow rules, but not deny rules\.
+ You can specify separate rules for inbound and outbound traffic\. 
+ When you create a security group, it has no inbound rules\. Therefore, no inbound traffic originating from another host to your instance is allowed until you add inbound rules to the security group\.
+ By default, a security group includes an outbound rule that allows all outbound traffic\. You can remove the rule and add outbound rules that allow specific outbound traffic only\. If your security group has no outbound rules, no outbound traffic originating from your instance is allowed\.
+ Security groups are stateful — if you send a request from your instance, the response traffic for that request is allowed to flow in regardless of inbound security group rules\. Responses to allowed inbound traffic are allowed to flow out, regardless of outbound rules\.
**Note**  
Some types of traffic are tracked differently to others\. For more information, see [Connection Tracking](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#security-group-connection-tracking) in the *Amazon EC2 User Guide for Linux Instances*\.
+ Instances associated with a security group can't talk to each other unless you add rules allowing it \(exception: the default security group has these rules by default\)\.
+ Security groups are associated with network interfaces\. After you launch an instance, you can change the security groups associated with the instance, which changes the security groups associated with the primary network interface \(eth0\)\. You can also change the security groups associated with any other network interface\. For more information about network interfaces, see [Elastic Network Interfaces](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html)\.
+ When you create a security group, you must provide it with a name and a description\. The following rules apply:
  + Names and descriptions can be up to 255 characters in length\.
  + Names and descriptions are limited to the following characters: a\-z, A\-Z, 0\-9, spaces, and \.\_\-:/\(\)\#,@\[\]\+=&;\{\}\!$\*\.
  + A security group name cannot start with `sg-`\.
  + A security group name must be unique within the VPC\.

## Default Security Group for Your VPC<a name="DefaultSecurityGroup"></a>

Your VPC automatically comes with a default security group\. Each EC2 instance that you launch in your VPC is automatically associated with the default security group if you don't specify a different security group when you launch the instance\.

The following table describes the default rules for a default security group\.


|  | 
| --- |
| Inbound | 
|  Source  |  Protocol  |  Port Range  |  Comments  | 
|  The security group ID \(sg\-*xxxxxxxx*\)  |  All  |  All  |  Allow inbound traffic from instances assigned to the same security group\.  | 
|   Outbound   | 
|  Destination  |  Protocol  |  Port Range  |  Comments  | 
|  0\.0\.0\.0/0  |  All  |  All  |  Allow all outbound IPv4 traffic\.  | 
| ::/0 | All | All | Allow all outbound IPv6 traffic\. This rule is added by default if you create a VPC with an IPv6 CIDR block or if you associate an IPv6 CIDR block with your existing VPC\.  | 

You can change the rules for the default security group\.

You can't delete a default security group\. If you try to delete the default security group, you'll get the following error: `Client.CannotDelete: the specified group: "sg-51530134" name: "default" cannot be deleted by a user`\. 

**Note**  
If you've modified the outbound rules for your security group, we do not automatically add an outbound rule for IPv6 traffic when you associate an IPv6 block with your VPC\.

## Security Group Rules<a name="SecurityGroupRules"></a>

You can add or remove rules for a security group \(also referred to as *authorizing* or *revoking* inbound or outbound access\)\. A rule applies either to inbound traffic \(ingress\) or outbound traffic \(egress\)\. You can grant access to a specific CIDR range, or to another security group in your VPC or in a peer VPC \(requires a VPC peering connection\)\.

The following are the basic parts of a security group rule in a VPC:
+ \(Inbound rules only\) The source of the traffic and the destination port or port range\. The source can be another security group, an IPv4 or IPv6 CIDR block, or a single IPv4 or IPv6 address\.
+ \(Outbound rules only\) The destination for the traffic and the destination port or port range\. The destination can be another security group, an IPv4 or IPv6 CIDR block, or a single IPv4 or IPv6 address\.
+ Any protocol that has a standard protocol number \(for a list, see [Protocol Numbers](http://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)\)\. If you specify ICMP as the protocol, you can specify any or all of the ICMP types and codes\.
+ An optional description for the security group rule to help you identify it later\. A description can be up to 255 characters in length\. Allowed characters are a\-z, A\-Z, 0\-9, spaces, and \.\_\-:/\(\)\#,@\[\]\+=;\{\}\!$\*\.

When you specify a CIDR block as the source for a rule, traffic is allowed from the specified addresses for the specified protocol and port\. When you specify a security group as the source for a rule, traffic is allowed from the elastic network interfaces \(ENI\) for the instances associated with the source security group for the specified protocol and port\. Adding a security group as a source does not add rules from the source security group\.

If you specify a single IPv4 address, specify the address using the /32 prefix length\. If you specify a single IPv6 address, specify it using the /128 prefix length\.

Some systems for setting up firewalls let you filter on source ports\. Security groups let you filter only on destination ports\.

When you add or remove rules, they are automatically applied to all instances associated with the security group\. 

The kind of rules you add can depend on the purpose of the instance\. The following table describes example rules for a security group for web servers\. The web servers can receive HTTP and HTTPS traffic from all IPv4 and IPv6 addresses, and send SQL or MySQL traffic to a database server\.


|  | 
| --- |
| Inbound | 
|  Source  |  Protocol  |  Port Range  |  Comments  | 
|  0\.0\.0\.0/0  |  TCP  |  80  |  Allow inbound HTTP access from all IPv4 addresses  | 
| ::/0 | TCP | 80 | Allow inbound HTTP access from all IPv6 addresses | 
|  0\.0\.0\.0/0  |  TCP  |  443  |  Allow inbound HTTPS access from all IPv4 addresses  | 
| ::/0 | TCP | 443 | Allow inbound HTTPS access from all IPv6 addresses | 
|  Your network's public IPv4 address range  |  TCP  |  22  |  Allow inbound SSH access to Linux instances from IPv4 IP addresses in your network \(over the Internet gateway\)  | 
|  Your network's public IPv4 address range  |  TCP  |  3389  |  Allow inbound RDP access to Windows instances from IPv4 IP addresses in your network \(over the Internet gateway\)  | 
|   Outbound   | 
|  Destination  |  Protocol  |  Port Range  |  Comments  | 
|  The ID of the security group for your database servers  |  TCP  |  1433  |  Allow outbound Microsoft SQL Server access to instances in the specified security group  | 
|  The ID of the security group for your MySQL database servers  |  TCP  |  3306  |  Allow outbound MySQL access to instances in the specified security group  | 

A database server would need a different set of rules; for example, instead of inbound HTTP and HTTPS traffic, you can add a rule that allows inbound MySQL or Microsoft SQL Server access\. For an example of security group rules for web servers and database servers, see [Security](VPC_Scenario3.md#VPC_Scenario3_Security)\. 

For examples of security group rules for specific kinds of access, see [Security Group Rules Reference](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules-reference.html) in the *Amazon EC2 User Guide for Linux Instances*\.

### Stale Security Group Rules<a name="vpc-stale-security-group-rules"></a>

If your VPC has a VPC peering connection with another VPC, a security group rule can reference another security group in the peer VPC\. This allows instances associated with the referenced security group to communicate with instances associated with the referencing security group\. 

If the owner of the peer VPC deletes the referenced security group, or if you or the owner of the peer VPC deletes the VPC peering connection, the security group rule is marked as `stale`\. You can delete stale security group rules as you would any other security group rule\.

For more information, see [Working With Stale Security Groups](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-security-groups.html#vpc-peering-stale-groups) in the *Amazon VPC Peering Guide*\.

## Differences Between Security Groups for EC2\-Classic and EC2\-VPC<a name="VPC_Security_Group_Differences"></a>

If you're already an Amazon EC2 user, you're probably familiar with security groups\. However, you can't use the security groups that you've created for use with EC2\-Classic with instances in your VPC\. You must create security groups specifically for use with instances in your VPC\. The rules you create for use with a security group for a VPC can't reference a security group for EC2\-Classic, and vice versa\.

The following table summarizes the differences between security groups for use with EC2\-Classic and those for use with EC2\-VPC\.


| EC2\-Classic | EC2\-VPC | 
| --- | --- | 
|  You can create up to 500 security groups per region\.  |  You can create up to 500 security groups per VPC\.  | 
|  You can add up to 100 rules to a security group\.  |  You can add up to 50 rules to a security group\.  | 
|  You can add rules for inbound traffic only\.  |  You can add rules for inbound and outbound traffic\.  | 
|  You can assign up to 500 security groups to an instance\.  |  You can assign up to 5 security groups to a network interface\.   | 
|  You can reference security groups from other AWS accounts\.  |  You can reference security groups from your VPC or from a peer VPC in a VPC peering connection only\. The peer VPC can be in a different account\.  | 
|  After you launch an instance, you can't change the security groups assigned to it\.  |  You can change the security groups assigned to an instance after it's launched\.  | 
|  When you add a rule to a security group, you don't have to specify a protocol, and only TCP, UDP, or ICMP are available\.  |  When you add a rule to a security group, you must specify a protocol, and it can be any protocol with a standard protocol number, or all protocols \(see [Protocol Numbers](http://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)\)\.  | 
|  When you add a rule to a security group, you must specify port numbers \(for TCP or UDP\)\.  |  When you add a rule to a security group, you can specify port numbers only if the rule is for TCP or UDP, and you can specify all port numbers\.  | 
| Security groups that are referenced in another security group's rules cannot be deleted\. | Security groups that are referenced in another security group's rules can be deleted if the security groups are in different VPCs\. If the referenced security group is deleted, the rule is marked as stale\. You can use the [describe\-stale\-security\-groups](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-stale-security-groups.html) AWS CLI command to identify stale rules\. | 
| You cannot specify an IPv6 CIDR block or an IPv6 address as the source or destination in a security group rule\. | You can specify an IPv6 CIDR block or an IPv6 address as the source or destination in a security group rule\. | 

## Working with Security Groups<a name="WorkingWithSecurityGroups"></a>

The following tasks show you how to work with security groups using the Amazon VPC console\.

**Topics**
+ [Modifying the Default Security Group](#ModifyingSecurityGroups)
+ [Creating a Security Group](#CreatingSecurityGroups)
+ [Adding, Removing, and Updating Rules](#AddRemoveRules)
+ [Changing an Instance's Security Groups](#SG_Changing_Group_Membership)
+ [Deleting a Security Group](#DeleteSecurityGroup)
+ [Deleting the 2009\-07\-15\-default Security Group](#DeleteSecurityGroup-2009)

### Modifying the Default Security Group<a name="ModifyingSecurityGroups"></a>

Your VPC includes a default security group whose initial rules are to deny all inbound traffic, allow all outbound traffic, and allow all traffic between instances in the group\. You can't delete this group; however, you can change the group's rules\. The procedure is the same as modifying any other security group\. For more information, see [Adding, Removing, and Updating Rules](#AddRemoveRules)\.

### Creating a Security Group<a name="CreatingSecurityGroups"></a>

Although you can use the default security group for your instances, you might want to create your own groups to reflect the different roles that instances play in your system\.

**To create a security group using the console**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Security Groups**\.

1. Choose **Create Security Group**\.

1. Enter a name of the security group \(for example, `my-security-group`\) and provide a description\. Select the ID of your VPC from the **VPC** menu and choose **Yes, Create**\.

**To create a security group using the command line**
+ [create\-security\-group](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html) \(AWS CLI\)
+ [New\-EC2SecurityGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2SecurityGroup.html) \(AWS Tools for Windows PowerShell\)

**Describe one or more security groups using the command line**
+ [describe\-security\-groups](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html) \(AWS CLI\)
+ [Get\-EC2SecurityGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2SecurityGroup.html) \(AWS Tools for Windows PowerShell\)

By default, new security groups start with only an outbound rule that allows all traffic to leave the instances\. You must add rules to enable any inbound traffic or to restrict the outbound traffic\.

### Adding, Removing, and Updating Rules<a name="AddRemoveRules"></a>

When you add or remove a rule, any instances already assigned to the security group are subject to the change\. 

If you have a VPC peering connection, you can reference security groups from the peer VPC as the source or destination in your security group rules\. For more information, see [Updating Your Security Groups to Reference Peered VPC Security Groups](https://docs.aws.amazon.com/vpc/latest/peering/working-with-vpc-peering.html#vpc-peering-security-groups) in the *Amazon VPC Peering Guide*\.

**To add a rule using the console**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Security Groups**\.

1. Select the security group to update\. The details pane displays the details for the security group, plus tabs for working with its inbound rules and outbound rules\.

1. On the **Inbound Rules** tab, choose **Edit**\. Select an option for a rule for inbound traffic for **Type**, and then fill in the required information\. For example, for a public web server, choose **HTTP** or **HTTPS** and specify a value for **Source** as `0.0.0.0/0`\. 
**Note**  
If you use `0.0.0.0/0`, you enable all IPv4 addresses to access your instance using HTTP or HTTPS\. To restrict access, enter a specific IP address or range of addresses\.

1. Optionally provide a description for the rule, and choose **Save**\.

1. You can also allow communication between all instances associated with this security group\. On the **Inbound Rules** tab, choose **All Traffic** from the **Type** list\. Start typing the ID of the security group for **Source**; this provides you with a list of security groups\. Select the security group from the list and choose **Save**\. 

1. If you need to, you can use the **Outbound Rules** tab to add rules for outbound traffic\.

**To delete a rule using the console**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Security Groups**\.

1. Select the security group to update\. The details pane displays the details for the security group, plus tabs for working with its inbound rules and outbound rules\.

1. Choose **Edit**, select the role to delete, and then choose **Remove**, **Save**\. 

When you modify the protocol, port range, or source or destination of an existing security group rule using the console, the console deletes the existing rule and adds a new one for you\. 

**To update a rule using the console**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Security Groups**\.

1. Select the security group to update, and choose **Inbound Rules** to update a rule for inbound traffic or **Outbound Rules** to update a rule for outbound traffic\.

1. Choose **Edit**\. Modify the rule entry as required and choose **Save**\.

To update the protocol, port range, or source or destination of an existing rule using the Amazon EC2 API or a command line tool, you cannot modify the rule; instead, you must delete the existing rule and add a new rule\. To update the rule description only, you can use the [update\-security\-group\-rule\-descriptions\-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/update-security-group-rule-descriptions-ingress.html) and [update\-security\-group\-rule\-descriptions\-egress](https://docs.aws.amazon.com/cli/latest/reference/ec2/update-security-group-rule-descriptions-egress.html) commands\. 

**To add a rule to a security group using the command line**
+ [authorize\-security\-group\-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html) and [authorize\-security\-group\-egress](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-egress.html) \(AWS CLI\)
+ [Grant\-EC2SecurityGroupIngress](https://docs.aws.amazon.com/powershell/latest/reference/items/Grant-EC2SecurityGroupIngress.html) and [Grant\-EC2SecurityGroupEgress](https://docs.aws.amazon.com/powershell/latest/reference/items/Grant-EC2SecurityGroupEgress.html) \(AWS Tools for Windows PowerShell\)

**To delete a rule from a security group using the command line**
+ [revoke\-security\-group\-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/revoke-security-group-ingress.html) and [revoke\-security\-group\-egress](https://docs.aws.amazon.com/cli/latest/reference/ec2/revoke-security-group-egress.html)\(AWS CLI\)
+ [Revoke\-EC2SecurityGroupIngress](https://docs.aws.amazon.com/powershell/latest/reference/items/Revoke-EC2SecurityGroupIngress.html) and [Revoke\-EC2SecurityGroupEgress](https://docs.aws.amazon.com/powershell/latest/reference/items/Revoke-EC2SecurityGroupEgress.html) \(AWS Tools for Windows PowerShell\)

**To update the description for a security group rule using the command line**
+ [update\-security\-group\-rule\-descriptions\-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/update-security-group-rule-descriptions-ingress.html) and [update\-security\-group\-rule\-descriptions\-egress](https://docs.aws.amazon.com/cli/latest/reference/ec2/update-security-group-rule-descriptions-egress.html) \(AWS CLI\)
+ [Update\-EC2SecurityGroupRuleIngressDescription](https://docs.aws.amazon.com/powershell/latest/reference/items/Update-EC2SecurityGroupRuleIngressDescription.html) and [Update\-EC2SecurityGroupRuleEgressDescription](https://docs.aws.amazon.com/powershell/latest/reference/items/Update-EC2SecurityGroupRuleEgressDescription.html) \(AWS Tools for Windows PowerShell\)

### Changing an Instance's Security Groups<a name="SG_Changing_Group_Membership"></a>

After you launch an instance into a VPC, you can change the security groups that are associated with the instance\. You can change the security groups for an instance when the instance is in the `running` or `stopped` state\.

**Note**  
This procedure changes the security groups that are associated with the primary network interface \(eth0\) of the instance\. To change the security groups for other network interfaces, see [Changing the Security Group of a Network Interface](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#eni_security_group)\.

**To change the security groups for an instance using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Open the context \(right\-click\) menu for the instance and choose **Networking**, **Change Security Groups**\. 

1. In the **Change Security Groups** dialog box, select one or more security groups from the list and choose **Assign Security Groups**\.

**To change the security groups for an instance using the command line**
+ [modify\-instance\-attribute](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-attribute.html) \(AWS CLI\)
+ [Edit\-EC2InstanceAttribute](https://docs.aws.amazon.com/powershell/latest/reference/items/Edit-EC2InstanceAttribute.html) \(AWS Tools for Windows PowerShell\)

### Deleting a Security Group<a name="DeleteSecurityGroup"></a>

You can delete a security group only if there are no instances assigned to it \(either running or stopped\)\. You can assign the instances to another security group before you delete the security group \(see [Changing an Instance's Security Groups](#SG_Changing_Group_Membership)\)\. You can't delete a default security group\.

If you're using the console, you can delete more than one security group at a time\. If you're using the command line or the API, you can only delete one security group at a time\.

**To delete a security group using the console**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Security Groups**\.

1. Select one or more security groups and choose **Security Group Actions**, **Delete Security Group**\.

1. In the **Delete Security Group** dialog box, choose **Yes, Delete**\.

**To delete a security group using the command line**
+ [delete\-security\-group](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-security-group.html) \(AWS CLI\)
+ [Remove\-EC2SecurityGroup](https://docs.aws.amazon.com/powershell/latest/reference/items/Remove-EC2SecurityGroup.html) \(AWS Tools for Windows PowerShell\)

### Deleting the 2009\-07\-15\-default Security Group<a name="DeleteSecurityGroup-2009"></a>

Any VPC created using an API version older than 2011\-01\-01 has the `2009-07-15-default` security group\. This security group exists in addition to the regular `default` security group that comes with every VPC\. You can't attach an Internet gateway to a VPC that has the `2009-07-15-default` security group\. Therefore, you must delete this security group before you can attach an Internet gateway to the VPC\.

**Note**  
If you assigned this security group to any instances, you must assign these instances a different security group before you can delete the security group\.

**To delete the `2009-07-15-default` security group**

1. Ensure that this security group is not assigned to any instances\.

   1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

   1. In the navigation pane, choose **Network Interfaces**\.

   1. Select the network interface for the instance from the list, and choose **Change Security Groups**, **Actions**\. 

   1. In the **Change Security Groups** dialog box, select a new security group from the list, and choose **Save**\.
**Note**  
When changing an instance's security group, you can select multiple groups from the list\. The security groups that you select replace the current security groups for the instance\.

   1. Repeat the preceding steps for each instance\.

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Security Groups**\.

1. Choose the `2009-07-15-default` security group, then choose **Security Group Actions**, **Delete Security Group**\.

1. In the **Delete Security Group** dialog box, choose **Yes, Delete**\.