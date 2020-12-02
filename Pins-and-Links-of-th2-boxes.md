### Pins.
Each box will have a number of connectors called pins. The pins are used by the box to send/receive messages or to execute gRPC commands. In the example below, the box has two pins. Each pin have a name (it is used in the links), a connection-type (MQ or gRPC) and attributes. 

Attributes describe what message stream goes through this particular pin. They are specific for each box and here, for instance, two pins – in and in_raw – are defined. They describe the raw and the parsed messages that comes into the box from the environment under test. Some attributes are optional while one of them is mandatory.

```yaml
    - name: in
      connection-type: mq
      attributes:
        - first
        - parsed
        - publish
        - store
    - name: in_raw
      connection-type: mq
      attributes:
        - first
        - raw
        - publish
        - store
```
If you are defining a pin to which data will be published by the current box, you must specify the "publish" attribute; if the pin is supposed to receive data from another box, then you can optionally specify "subscribe". Although the "subscribe" attribute is optional, it’s still recommended to specify it, to maintain consistency. If the pin is accepting data and the "subscribe" attribute is not specified, then by default the pin will be considered as "subscribe" anyway. You cannot apply both attributes to one pin at the same time. A pin can have either a "publish" or a "subscribe" attribute.

Additionally, a pin can have a filter section. In this case, a message is sent via this particular pin only if this message satisfies the filter parameter (refer to the example below).  
```yaml
- name: fix_to_send
      connection-type: mq
      attributes:
        - send
        - parsed
        - subscribe
      filters:
        - metadata:
            - field-name: session_alias
              expected-value: conn1_session_alias
              operation: EQUAL
```
Attributes: 
| Attribute name | Description | Used for |
| --- | --- | --- |
| first | Base attribute marks pin for transfer messages which sent from a server to a client. | conn component to retransmit dialog between conn and remote system into th2.|
| second | Base attribute marks pin for transfer messages which sent from a client to a server. | conn component to retransmit dialog between conn and remote system into th2. |
| oe | Special attribute to mark “Order entry“ data stream | act|
| publish | Base attribute marks pin which publish "messages" via MQ | Any components which publish "message" via MQ, for example:<ul><li>conn to codec</li><li>codec to act / check / …</li><li>act to conn</li><li>conn to estore</li><li>etc.</li></ul> |
| subscribe | Base attribute marks pin which subscribe to "messages" via MQ | Any components which publish "message" via MQ, for example:<ul><li>conn to codec</li><li>codec to act / check / …</li><li>act to conn</li><li>conn to estore</li><li>etc.</li></ul> |
| parsed | Base attribute marks pin which transfer message batches | Codec publish after decode and consume for encode it. act, check, script work with this type of “messages” |
| raw | Base attribute marks pin which transfer raw message batches | Codec publish after encode and consume for decode it. | Conn and read produce this type of “messages” |
| event | Base attribute marks pin which transfer event batches | Any box publishes events. Estore consume to this type of “messages” |
| store | Base attribute indicates that messages that comes in this pin will be stored in cradle. | Pins which produce data to th2, for example in conn, read, should be marks this attribute. |
| grpc_to_verifier | fake | act |
| grpc_to_act | fake | verifier |
| decoder_in | Describes input pin for codec decoder(transform protocol message to human-readable). | codec |
| decoder_out | Describes output pin for codec decoder(transform protocol message to human-readable). | codec |
| encoder_in | Describes input pin for codec encoder(transform human-readable message to protocol message). |codec |
| encoder_out | Describes output pin for codec encoder(transform human-readable message to protocol message). | codec |
| send | Special attribute for conn pin to receive data from act or other components | conn |

### Links.
After all the pins are defined and configured, you should also specify the links between them. It can be done by uploading a special CR called Th2Link. Based on what components links are connecting, they can be separated in several files(e.g. from-codec-links.yml, from-act-alinks.yml, dictionary-links.yml.) Also all links can be described in one file, but links for dictionary should be in `dictionaries-relation section`, and all other links in `boxes-relation` section.<br></br>
dictionary-links example: ([full](https://github.com/th2-net/th2-infra-schema-demo/blob/master/links/dictionary-links.yml))
```yaml
apiVersion: th2.exactpro.com/v1
kind: Th2Link
metadata:
  name: dictionary-links
spec:
  dictionaries-relation:
    - name: codec-fix-dictionary
      box: codec-fix
      dictionary:
        name: fix50-generic
        type: MAIN
```
boxes-links example: ([full](https://github.com/th2-net/th2-infra-schema-demo/blob/master/links/from-conn-links.yml))
```yaml
apiVersion: th2.exactpro.com/v1
kind: Th2Link
metadata:
  name: from-conn-links
spec:
  boxes-relation:
    router-mq:
      - name: democonn1-codec
        from:
          box: demo-conn1
          pin: in_raw
        to:
          box: codec-fix
          pin: in_codec_decode
```
MQ links are described in the “router-mq” section. When connecting MQ links, you should keep in mind that the pins that are marked with the "publish" attribute must be specified in the "from" section, and those marked with "subscribe" (or not marked with either) must be specified in the "to" section.
Each link has the name and the pin of the box from which this link goes, and the name and the pin of the box to which it leads.
gRPC links are described in the section “router-grpc”. <br></br>
grpc-link example: ([full](https://github.com/th2-net/th2-infra-schema-demo/blob/master/links/from-act-links.yml))
```yaml
apiVersion: th2.exactpro.com/v1
kind: Th2Link
metadata:
  name: from-act-links
spec:
  boxes-relation:
    router-grpc:
      - name: act-to-check1
        from:
          service-class: com.exactpro.th2.check1.Check1Handler
          strategy: filter
          box: act-fix
          pin: to_check1
        to:
          service-class: com.exactpro.th2.check1.grpc.Check1Service
          strategy: robin
          box: check1
          pin: server
```