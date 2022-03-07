---
layout: page
title: "Azure Terraform Notes"
permalink: /azure/terraform
---

<p>Useful notes on variables:</p>
<ul>
  <li><a href="https://upcloud.com/community/tutorials/terraform-variables/" target="#">Upcloud</a></li>
  <li><a href="https://dev.to/pwd9000/terraform-complex-variable-types-173e" target="#">Dev Community</a></li>
</ul>

<h1>Contents</h1>
* TOC
{:toc}
# Terraform Architecture 
<a href="#markdown-toc">Top</a>
<p>Terraform is a tool to consume upstream APIs. The Azure REST API exposes a client library that 
encapsulates the request-response objects of the API. The Terraform Azure provider uses this 
client library to connect and invoke remote API endpoints for Azure. Each provider is a plugin
that be added to the Terraform core, allowing Terraform to interact with multiple upstream APIs</p>
<p>Terraform maintains a state file that is created the first time a Terraform configuration is 
executed in a working directory. The state file is a JSON representation of the remote
infrastructure and is used by Terraform to plan and execute the changes to be made to the 
infrastructure when Terraform commands are run.</p>

# Terraform Blocks 
<a href="#markdown-toc">Top</a>

<p>HCL scripts are used to declare the desired state using a series of code blocks. Each block has a type and serves a specific purpose:</p>

## terraform
<p>The terraform block is used to configure the terraform version, which providers will be used and the backend for the configuration</p>
<pre><code>
terraform {
  required_providers {
    azurerm = {
      version = "=2.86"
      source = "hashicorp/azurerm"
    }
  }

  backend azurerm {}
}
</code></pre>

## provider
<p>Used to configure provider plugins:</p>
<pre><code>
provider "azurerm" {
  features {}
}
</code></pre>

## variable
<p>Used to accept external values as parameters to be used by all other blocks, apart from the terraform block</p>

## resource
<p>Used to represent a resource instance in the remote infrastructure that will be managed by terraform. The resource block defines the desired resource configuration</p>

## output
<p>Used to get information about the infrastructure after deployment. The 'terraform output' 
command can also be used to retrieve the value of an output variable. The variable value can 
be retrieved directly or as part of an expression:</p>
<pre>
variable "compass" {
  type = string
  default = "West"
}

output "direction" {
  value = var.compass
}

output "region" {
  value = "UK ${var.compass}"
}
</pre>

## data
<p>blank for now</p>

## module
<p>blank for now</p>

## locals
<p>blank for now</p>

# Terraform Workflow 
<a href="#markdown-toc">Top</a>

<p>The general workflow for deploying infrastructure in Terraform is:</p>
<ol>
  <li>Initialise the working directory (terraform init)</li>
  <li>Validate the scripts (terraform validate)</li>
  <li>Run the plan (terraform plan)</li>
  <li>Deploy the infrastructure (terraform apply)</li>
  <li>Destroy the infrastructure (terraform destroy</li>
</ol>
<p>Terraform 'init' is executed in the directory containing the Terraform scripts 
and will initialise the state file and download required providers and modules 
defined by the scripts. Modules and providers are downloaded and stored in the 
hidden '.terraform' folder in the working directory.</p>
<p>Terraform 'validate' can be used to validate the syntax of the scripts in the 
working directory: there is also a 'fmt' command available to apply best 
practice formatting to the scripts</p>
<p>The 'terraform plan' command is used to determine what changes will be 
made when the scripts are applied. It compares the target infrastructure to the 
current state file and then compares the declared configuration to the state 
file. The difference between the two allows terraform to determine what changes
need to be made when the configuration is applied and to output these changes as
a plan</p>
<p>After reviewing the plan, 'terraform apply' can be used to apply the 
desired configuration as defined in the scripts to the target infrastructure. 
If the apply is successful, Terraform generates a state file which, by default, 
is strored in the current working directory as 'terraform.tfstate'.</p>
<p>The 'terraform destroy' command can be used to remove resources from 
environment and also from the state file. The '-target' option will allow 
you to specify a specific resource to remove:</p>
<pre>
terraform destroy -target azurerm_virtual_machine.user_vm[1]
</pre>

# Terraform Variables 
<a href="#markdown-toc">Top</a>

<p>Terraform variable blocks allow an existing configuration to be tailored 
using external input values. A variable block starts with the keyword 'variable' 
followed by the variable name and a block of attributes associated to the variable.
These attributes include:</p>
<ul>
  <li>type ("string", "bool", "number", "object", "map", "list", "tuple", "sets", "any")</li>
  <li>default: a default value to use if none is supplied</li>
  <li>description: a text string explaining the use of the variable</li>
  <li>validation: a block with code used to validate the value supplied for the variable</li>
  <li>sensitive: a boolean value to determine if the variable value is displayed by the Terraform CLI</li>
</ul>
<p>Variables and their default values can be declared within the same scripts that you use to define
your infrastructure code. However, it is often preferrable to create a separate 'variables.tf' for 
this purpose</p>
<p>Variables values can be assigned on the command-line:</p>
<pre>terraform plan -var azure_region="UK South"</pre>
<p>This becomes impractical if you want to override several default values. Instead you can 
additionally create a 'variable definitions' file. The variable definitions file should consist of only variable assignments:</p>
<pre>
azure_region = "West Europe"
list_of_fruit = ["mango", "pineapple", "blueberry", "grape"]
res_tags = {
    location   = "UK South"
    department = "Accounts"
    owner      = "Freda Perry"
}
</pre>
<p>Terraform will automatically load variable defintion files named either 'terraform.tfvars' or 
files ending in '.auto.tfvars'. If you prefer to use JSON format for your variable definition files, 
Terraform will also load any files named 'terraform.tfvars.json' or ending in 'auto.tfvars.json'.</p> 
<pre>
{
  "azure_region" : "West Europe",
  "list_of_fruit" : ["mango", "pineapple", "blueberry", "grape"],
  "res_tags" : {
    "location"   : "UK South",
    "department" : "Accounts",
    "owner"      : "Freda Perry"
  }
}
</pre>
<p>Variables can also assigned in environment variables by prefixing the variable name with "TF_VAR_":</p>
<pre>
export TF_VAR_LOCATION="UK South"
</pre>
<p>The corresponding variable "LOCATION" will need to be declared in your scripts to be accessible:</p>
<pre>
variable "LOCATION" {
  default = ""
}

output "region" {
  value = var.LOCATION
}
</pre>


## String Variables
<p>Values for string variables are assigned with double-quoted strings:</p>
<pre>
variable "azure_region" {
  type        = string
  default     = "UK West"
  description = "The Azure region to deploy resources"
}

output "az_region" {
  value = var.azure_region
}
</pre>
## List Variables
<p>Lists are assigned as comma-separated lists inside square brackets:</p>
<pre>
variable "list_of_fruit" {
  type    = list(string)
  default = ["apple", "pear", "orange", "peach"]
}

output "first_fruit" {
  value = var.list_of_fruit[0]
}

output "all_fruits" {
  value = var.list_of_fruit
}
</pre>

## Map Variables
<p>Map variables are collection types, and contain key-value pairs. The values should adhere to the 
declared datatype. Each key should be unique, where the keys and values are of type string:</p>
<pre>
variable "res_tags" {
  type       = map(string)
  default = {
    location   = "UK West"
    department = "Finance"
    owner      = "Fred Perry"
  }
}

output "this_department" {
  value = var.res_tags["department"]
}
</pre>

## Object Variables
<p>Object variables are collection types, with key-value pairs where the values for each key can 
be of different types:</p>
<pre>
variable "user_account" {
  type = object({
    logon    = string
    email    = string
    roles    = list(string)
    projects = map(string)
    enabled  = bool
  })
  default = {
    logon = "fred001"
    email = "fred@example.com"
    roles = ["user", "report writer", "remote access", "acr pull"]
    projects = {
      primary = "solaris"
      second  = "aix"
      third   = "rhel"
    }
    enabled = true
  }
}

output "user_details" {
  value = var.user_account.logon
}

output "main_project" {
  value = var.user_account.projects.primary
}
</pre>

## Set Variables
<p>Sets are collections that can contain only unique values of the same datatype:</p>
<pre>
# Although the value "two" is included twice in the default assignment
# the value "two" will only occur once in the actual set variable
variable "myset" {
  type    = set(string)
  default = ["one", "two", "three", "four", "two"]
}

output "set_members" {
  value = var.myset
}
</pre>
<p>Elements of a set are identified only by their value and don't have any index or key to select
with, so it is only possible to perform operations across all elements of the set</p>

## Tuple Variables
<p>Tuples are similar to lists but each value in the list can of a different type:</p>
<pre>
variable "user_info" {
  type    = tuple([string, bool, number])
  default = ["Finance", true, 1]
}

output "tuple_values" {
  value = var.user_info
}

output "user_enabled" {
  value = var.user_info[1]
}
</pre>

# Remote State 
<a href="#markdown-toc">Top</a>
<p>Terraform documentation on <cite><a href="https://www.terraform.io/language/settings/backends/azurerm">Terraform Backend</a></cite></p>
<p>Microsoft documentation on <cite><a href="https://docs.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=azure-cli" target="#">Remote State Storage</a></cite> for terraform</p>
<p>Storing Terraform state remotely allows multiple developers to work on the same
environment from different machines. Terraform state can be stored in Azure and then
referenced in the backend configuration of your Terraform scripts. Public access is allowed
to Azure storage to store Terraform state. To use remote
state in Azure Storage, configure the backend block of your terraform block with:</p>
<ul>
  <li>resource_group_name</li>
  <li>storage_account_name</li>
  <li>container_name</li>
  <li>key - the name of the state store file to be created</li>
</ul>
<pre><code>
terraform {
  required_version = "~>v1.1.5"
  required_providers {
    azurerm = {
      version = "~>2.36"
      source  = "hashicorp/azurerm"
    }
  }
  backend "azurerm" {
    resource_group_name  = "rg-remotestate"
    storage_account_name = "stgremotestate"
    container_name       = "statefiles"
    key                  = "project1.terraform.tfstate"
  }
}
</code></pre>

# Terraform Meta-Elements
<a href="#markdown-toc">Top</a>
<h2>Count</h2>
<p>The 'count' attribute can be added to any resource and the value of this 
attribute determines the number of resource instances to be deployed. The 
'count' attribute provides and 'index' property which can be use with 
variable interpolation to assign unique names to each resource created. The 
index begins at 0.</p>
<pre><code>
resource "azurerm_virtual_machine" "user_vm" {
  count = 3
  name = "${var.vm_name}_0${count.index}"
...
...
}

output vm_ids {
  value = azurerm_virtual_machine.user_vm[*].id
}
</code></pre>
<p>Terraform stores the list of resources created as an array in the terraform
state and the splatting operator '*' can be used to refer to this array as a 
whole.</p>
<p>The 'count' meta-element can also be used with a list variable, allowing more 
tailored names to be given to the resources created:</p>
<pre><code>
variable "vm_names" {
  type = list(string)
  default = ["vm_harry", "vm_larry", "vm_barry"]
}

resource "azurerm_virtual_machine" "user_vm" {
  count = length(var.vm_names)
  name = "${var.vm_names[count.index]}"
...
...
}

output vm_ids {
  value = azurerm_virtual_machine.user_vm[*].id
}
</code></pre>

<h2>For Each</h2>
<p>The 'for_each' meta element can be used to iterate a map or set 
to provision multiple resources from a single 
resource block. 'for_each' provides a 'each.key' and an 'each.value' attribute
to further tailor the properties of the resources created:</p>
<pre><code>
variable "user_machines" = {
  type = map(string)
  default = {
    "vm_harry" : "rg-env-dev",
    "vm_larry" : "rg-env-live",
    "vm_barry" : "rg-env-test"
  }
}

resource azurerm_virtual_machine "user_vm" {
  for_each = var.user_machines
  name = each.key
  resource_group_name = each.value
...
...
}
</code></pre>

<h2>Ternary Operator</h2>
<p>Terraform provides the traditional ternary operator '?:' to implement
conditional logic within blocks:</p>
<pre><code>
variable "user_machines" = {
  type = map(string)
  default = {
    "vm_harry" : "rg-env-dev",
    "vm_larry" : "rg-env-live",
    "vm_barry" : "rg-env-test"
  }
}

resource azurerm_virtual_machine "user_vm" {
  for_each = var.user_machines
  name = each.key
  resource_group_name = each.value
  vm_size = each.key == "vm_larry" ? "Standard_DS1_v2" : "Standard_FS1_v2"
...
...
}
</code></pre>
