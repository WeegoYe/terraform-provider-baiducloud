# Terraform docs generator

## Why

经过观察，大部分友商的 Terraform Plugins 文档都是人肉编写的，它们或多或少都有这样的问题：

* 风格难以统一，比如：章节顺序、空行数量、命名风格在不同产品之间有明显差异
* 细节存在问题，比如：空格数量、缩进多少、中下划线出现不统一，甚至还夹带中文符号
* 内容存在问题，比如：参数必须或选填跟代码不一致，列表中漏写参数或属性

然而，最大的问题是写文档需要消耗大量的时间和精力去整理内容整理格式，最后还发现总有这样那样的问题，甚至有时还会文档更新不及时。
机器可以一如始终，完全无误差地完成各项有规律的重复性工作，而且不会出错，自动地生成文档也是 golang 所推动的标准做法。

## How

Terraform Plugins 文档，不管是 resource 还是 data_source，主要都分以下这几个主题：

* name
* description
* example usage
* argument reference
* attributes reference

### name

name 是 resource 及 data_source 的完整命名，它来源于 Provider 的 DataSourcesMap(data_source) 及 ResourcesMap(resource) 定义。
例如以下 DataSourcesMap 中的 baiducloud_instance 与 baiducloud_vpc 就是一个标准的 name(for resource or data_source)：

```go
DataSourcesMap: map[string]*schema.Resource{
    "baiducloud_instance": dataSourceBaiduCloudInstance(),
    "baiducloud_vpc": dataSourceBaiduCloudVpc(),
}
```

### description & example usage

description 包括一个用于表头的一句话描述，与一个用于正文的详细说明。
example usage 则是一个或几个使用示例。

description & example usage 需要在对应 resource 及 data_source 定义的文件中出现，它是符合 golang 标准文档注释的写法。例如：

    /*
    Use this data source to get information about a BCC instance.
    \n
    ~> **NOTE:** The terminate operation of bcc does NOT take effect immediately，maybe takes for several minites.
    \n
    Example Usage
    \n
    ```hcl
    data "baiducloud_instance" "my-server"{
      image_id = "m-DpgNg8lO"
      name = "from-terraform"
      availability_zone = "cn-bj-c"
    }
    ```
    */
    package baiducloud

以上注释的格式要求如下：

    /*
    一句话描述
    \n
    在一句话描述基础上的补充描述，可以比较详细地说明各项内容，可以有多个段落。
    \n
    Example Usage
    \n
    Example Usage 是 必须的，在 Example Usage 以下的内容都会当成 Example Usage 填充到文档中。
    */
    package baiducloud

符合以上要求的注释将会自动提取并填写到文档中的对应位置。

### argument reference & attributes reference

Terraform 用 schema.Schema 来描述 argument reference & attributes reference，每个 schema.Schema 都会有一个 Description 字段。
如果 Description 的内容不为空，那么这个 schema.Schema 将会被认为是需要写到文档里面的，如果 Optional 或 Required 设置了，它会被认为是一个参数，如果 Computed 为 true 则认为是一个属性。例如：

#### argument

```go
map[string]*schema.Schema{
    "name": {
        Type:        schema.TypeString,
        Description: "Instance Name",
        Required:    true,
    },
}
```

#### attributes

```go
map[string]*schema.Schema{
    "availability_zone": {
        Type:        schema.TypeString,
        Description: "Availability Zone",
        Optional:    true,
        ForceNew:    true,
        Computed:    true,
    },
}
```

#### attributes list

属性中 Type 为 schema.TypeList 的 schema.Schema 也是支持的，它会被认为是一个列表，里面的子 schema.Schema 会依次列出填充到文档中。

```go
map[string]*schema.Schema{
    "instance_list": {
        Type:     schema.TypeList,
        Computed: true,
        Description: "A list of instances. Each element contains the following attributes:",
        Elem: &schema.Resource{
                Schema: map[string]*schema.Schema{
                "image_id": {
                    Type:        schema.TypeString,
                    Description: "Image ID",
                    Required:    true,
                },
                "name": {
                    Type:        schema.TypeString,
                    Description: "Instance Name",
                    Required:    true,
                },
                "availability_zone": {
                    Type:        schema.TypeString,
                    Description: "Availability Zone",
                    Optional:    true,
                    ForceNew:    true,
                    Computed:    true,
                },
            }
        }
    }
}
```

## 文档索引更新

文档索引文件，即 website/baiducloud.erb 的更新数据来源于 provider.go 的文件注释。

完成了新的 Data Sources 或 Resources 后，需要更新 provider.go 的文件注释，格式可参考已有的 Data Sources 或 Resources。

### Data Sources

在注释中找到 Data Sources，在它的下面填写新的 Data Sources 名称，比如：baiducloud_instance，注意前面需要空两个空格。

例如：

```go
Data Sources
  baiducloud_vpcs
  baiducloud_subnets
  baiducloud_route_rules
```

### Resources

在注释的 Data Sources 段之后，直接添加新的 Resources 类并添加 Resources，也可以将 Resources 添加到已有的 Resources 类。

例如：

```go
BCC Resources
  baiducloud_instance
  baiducloud_security_group
  baiducloud_cds

VPC Resources
  baiducloud_vpc
  baiducloud_subnet
```
