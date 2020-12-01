This page contains the description of event content supported by **rpt-viewer**.

It is possible to save any data to the event body as there are no structure checks before it is stored. If **rpt-viewer** encounters an unknown event body, it will fallback to json text representation. The content will still be visible.

**Event body structure**

Event body is an ordered array of “payload” objects. You can combine different payload types in a single event body.
```java
[
  {...},
  {...},
  {...}
]
```

## Payload types

### Message

This payload produces a single line of text with data that will be displayed

```java
[
  {
    "type": "message",
    "data": "My message for report"
  }
]
```
### Table

This payload produces a simple table with arbitrary number of columns and rows

```java
[
  {
    "type": "table",
    "rows": [
      {
        "Column A": "Column / Row - A1",
        "Column B": "Column / Row - B1",
      },
      {
        "Column A": "Column / Row - A2",
        "Column B": "Column / Row - B2",
      }
    ]
  }
]
```

### Tree table

This payload can be used to display complex data structures e.g. input parameters. This type contains a field “name”, which is optional. If it’s not set or is equal to null, no table name will be displayed.

```java
[
  {
    "type": "treeTable",
    "name": "tableName", // this one is optional
    "rows": {
      "Row A with some custom name": {
        "type": "row",
        "columns": {
          "Column 1 with some custom name": "some text (A1)",
          "Column 2": "some text (A2)",
        }
      },
      "Row B with some other name": {
        "type": "collection",
        "rows": {
          "Row BA": {
            "type": "row",
            "columns": {
              "Column 1 with some custom name": "some text (BA1)",
              "Column 2": "some text (BA2)",
            }
          }
          "Row BB": {
            "type": "row",
            "columns": {
              "Column 1 with some custom name": "some text (BB1)",
              "Column 2": "some text (BB2)",
            }
          }
        }
      }
    }
  }
]
```

### Verification

This payload can be used to render a hierarchical table which contains information about the comparison of two messages.

```java
[
  {
    "type": "verification",
    "fields": {
      "Field A": {
        "type": "field",
        "operation": "NOT_EMPTY",
        "status": "PASSED",
        "key": false,
        "actual": "value",
        "expected": "*"
      },
      "Field B": {
        "type": "field",
        "operation": "EQUAL",
        "status": "PASSED",
        "key": true,
        "actual": "2531410",
        "expected": "2531410"
      },
      "Sub message A": {
        "type": "collection",
        "operation": "EQUAL",
        "key": false,
        "actual": "1",
        "expected": "1",
        "fields": {
            "Field C": {
            "type": "field",
            "operation": "NOT_EQUAL",
            "status": "FAILED",
            "key": false,
            "actual": "9",
            "expected": "9"
          }
        }
      }
    }
  }
]
```
---
Please also find examples on Python below:

### Table example (python):

```python
class TableComponent:

    def __init__(self, headers: list) -> None:
        self.type = "table"
        self.rows = []
        self.headers = headers

    def add_row(self, *args):
        row = {}
        for arg in args:
            row[self.headers[args.index(arg)]] = arg
        self.rows.append(row)


        table = store.TableComponent(['Name', 'Value'])
        table.add_row('Filed_1', value_1)
        table.add_row('Filed_2', value_2)
```