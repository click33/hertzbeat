---
id: extend-point  
title: Custom Monitoring  
sidebar_label: Custom Monitoring    
---
> HertzBeat has custom monitoring ability. You only need to configure two YML file to fit a custom monitoring type.  
> Custom monitoring currently supports [HTTP protocol](extend-http)，[JDBC protocol](extend-jdbc)(mysql,mariadb,postgresql...), [SSH protocol](extend-ssh), JMX protocol, SNMP protocol. And it will support more general protocols in the future.        

### Custom Steps  

In order to configure a custom monitoring type, you need to add and configure two YML file.
1. Monitoring configuration definition file named after monitoring type - eg：example.yml should be in the installation directory /hertzbeat/define/app/
2. Monitoring parameter definition file named after monitoring type - eg：example.yml should be in the installation directory /hertzbeat/define/param/
3. Restart hertzbeat system, we successfully fit a new custom monitoring type.  

------- 
Configuration usages of the two files are detailed below.

### Monitoring configuration definition file   

> Monitoring configuration definition file is used to define *the name of monitoring type(international), request parameter mapping, index information, collection protocol configuration information*, etc.  

eg：Define a custom monitoring type named example which use the HTTP protocol to collect data.    
The file name: example.yml in /define/app/example.yml   

```yaml
# The monitoring type category：service-application service monitoring db-database monitoring custom-custom monitoring os-operating system monitoring
category: custom
# Monitoring application type(consistent with the file name) eg: linux windows tomcat mysql aws...
app: example
name:
  zh-CN: 模拟应用类型
  en-US: EXAMPLE APP
# parameter mapping map. These are input parameter variables which can be written to the configuration in form of ^_^host^_^. The system automatically replace variable's value.
# type means parameter type: 0-number number, 1-string cleartext string, 2-secret encrypted string
# required parameters - host
configmap:
  - key: host
    type: 1
  - key: port
    type: 0
  - key: username
    type: 1
  - key: password
    type: 2
# Metric group list
metrics:
# The first monitoring Metric group cpu
# Note：: the built-in monitoring Metrics have (responseTime - response time)
  - name: cpu
    # The smaller Metric group scheduling priority(0-127), the higher the priority. After completion of the high priority Metric group collection,the low priority Metric group will then be scheduled. Metric groups with the same priority  will be scheduled in parallel.
    # Metric group with a priority of 0 is an availability group which will be scheduled first. If the collection succeeds, the  scheduling will continue otherwise interrupt scheduling.
    priority: 0
    # Specific monitoring Metrics in the Metric group
    fields:
      # Metric information include field: name   type: field type(0-number: number, 1-string: string)   instance: primary key of instance or not   unit: Metric unit
      - field: hostname
        type: 1
        instance: true
      - field: usage
        type: 0
        unit: '%'
      - field: cores
        type: 0
      - field: waitTime
        type: 0
        unit: s
# (optional)Monitoring Metric alias mapping to the Metric name above. The field used to collect interface data is not the final Metric name directly. This alias is required for mapping conversion.
    aliasFields:
      - hostname
      - core1
      - core2
      - usage
      - allTime
      - runningTime
# (optional)The Metric calculation expression works with the above alias to calculate the final required Metric value.
# eg: cores=core1+core2, usage=usage, waitTime=allTime-runningTime
    calculates:
      - hostname=hostname
      - cores=core1+core2
      - usage=usage
      - waitTime=allTime-runningTime
# protocol for monitoring and collection  eg: sql, ssh, http, telnet, wmi, snmp, sdk
    protocol: http
# Specific collection configuration when the protocol is HTTP protocol 
    http:
      # host: ipv4 ipv6 domain name 
      host: ^_^host^_^
      # port
      port: ^_^port^_^
      # url request interface path 
      url: /metrics/cpu
      # request mode  GET POST PUT DELETE PATCH
      method: GET
      # enable ssl/tls or not, tthat is to say, HTTP or HTTPS. The default is false
      ssl: false
      # request header content 
      headers:
        apiVersion: v1
      # request parameter content 
      params:
        param1: param1
        param2: param2
      # authorization 
      authorization:
        # authorization method : Basic Auth, Digest Auth, Bearer Token
        type: Basic Auth
        basicAuthUsername: ^_^username^_^
        basicAuthPassword: ^_^password^_^
      # parsing method for reponse data: default-system rules, jsonPath-jsonPath script, website-website availability Metric monitoring 
      # todo xmlPath-xmlPath script,prometheus-Prometheus data rules
      parseType: jsonPath
      parseScript: '$'

  - name: memory
    priority: 1
    fields:
      - field: hostname
        type: 1
        instance: true
      - field: total
        type: 0
        unit: kb
      - field: usage
        type: 0
        unit: '%'
      - field: speed
        type: 0
    protocol: http
    http:
      host: ^_^host^_^
      port: ^_^port^_^
      url: /metrics/memory
      method: GET
      headers:
        apiVersion: v1
      params:
        param1: param1
        param2: param2
      authorization:
        type: Basic Auth
        basicAuthUsername: ^_^username^_^
        basicAuthPassword: ^_^password^_^
      parseType: default
```

### Monitoring parameter definition file

> Monitoring parameter definition file is used to define *required input parameter field structure definition (Front-end page render input parameter box according to structure)*.   

eg：Define a custom monitoring type named example which use the HTTP protocol to collect data.    
The file name: example.yml in /define/param/example.yml   

```yaml
# Monitoring application type name(consistent with the file name) eg: linux windows tomcat mysql aws...
app: example
# required parameters - host(ipv4, ipv6, domain name)
param:
    # field-field name identifier 
  - field: host
    # name-parameter field display name 
    name: 
      zh-CN: 主机Host
      en-US: Host
    # type-field type, style(most mappings are input label type attribute)
    type: host
    # required or not  true-required  false-optional
    required: true
  - field: port
    name: 
      zh-CN: 端口
      en-US: Port
    type: number
    # When type is number, range is used to represent the range.
    range: '[0,65535]'
    required: true
    # port default
    defaultValue: 80
    # Prompt information of parameter input box 
    placeholder: 'Please enter the port'
  - field: username
    name: 
      zh-CN: 用户名
      en-US: Username
    type: text
    # When type is text, use limit to indicate the string limit size
    limit: 20
    required: false
  - field: password
    name: 
      zh-CN: 密码
      en-US: Password
    type: password
    required: false
  - field: ssl
    name: 
      zh-CN: 启动SSL
      en-US: Enable SSL
    # When type is boolean, front end uses switch to show the switch
    type: boolean
    required: false
  - field: method
    name: 
      zh-CN: 请求方式
      en-US: Method
    type: radio
    required: true
    # When type is radio or checkbox, option indicates the list of selectable values {name1:value1,name2:value2}
    options:
      - label: GET request
        value: GET
      - label: POST request
        value: POST
      - label: PUT request
        value: PUT
      - label: DELETE request
        value: DELETE
```
