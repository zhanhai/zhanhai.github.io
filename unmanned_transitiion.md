# Task Generator

## Problem

> Given an initial state and final state, generate a sequence of actions to convert the initial state to the required final state.

## Algorithm

1. Analyse the initial state and the expected state.
2. Analyse contraint if there is any.
3. Compare the object list of the two states and get required actions for the transition.
4. Build task sequence by analysing dependenc relation of them .

Let's go over the details with an example.

## Example

### 1. Analyse the initial and final states

Given the two states:

```json
init state
{
    "vss":
    [
        {
            "key": "vss1",
            "nic":
            [
                "n1",
                "n2"
            ],
            "configStatus": "green"
        }
    ]
}
```

```json
final state: remove vss1
{
    "vss":
    [
        {
            "key": "vss2",
            "nic":
            [
            	"n1",
            	"n2"
            ],
            "configStatus": "green"
        }
    ]
}
```

Before we could analyse, we need some knowledge about objects, such as:

> * `vss` indicates a list of VSS resource objects. `key` fild of each elemnt in the list is the id field of VSS object.
> * `nic` indicates a list of NIC resource objects. Each element in the `nic` list is id of a NIC resoruce object.
> * `configStatus` is an attribute object. It's owner is VSS resource object.
> * `nic` object contained in `vss` object is a `bind` relation between the two objects.

With the preceding knowledge(a.k.a `meta data`), the analysis result could be:

```javascript
var initialState = {
      resource(kind=vss, id=vss1),
      resource(kind=nic, id=n1),
      resource(kind=nic, id=n2),
      attribute(name=configStatus, owner=vss1, value=green),
      association(kind=bind, parent=vss1, child=n1),
      association(kind=bind, parent=vss1, child=n2)
};

var finalState = {
      resource(kind=vss, id=vss2),
      resource(kind=nic, id=n1),
      resource(kind=nic, id=n2),
      attribute(name=configStatus, owner=vss2, value=green),
      association(kind=bind, parent=vss2, child=n1),
      association(kind=bind, parent=vss2, child=n2)
}
```

### 2. Analyse constraint

Constraint can be expressed as an expression which can be evaluated against state.

In our case, we want that there is at least one vss stay connected over the process. Such condition can be defined in following expression:

```javascript
$.associations
      .filter({kind=bind})
      .size() > 0
```

>`$` denotes the current state(a list of objects).
>
>`$.associations` is a list of all association objects in current state.

### 3. Produce difference between the two states

Since state is actually a list of objects, so comparision ends with list of removal and creation actions of objects.

```javascript
var changes = [
     remove(kind=vss, id=vss1),
     unset(owner=vss1, name=configStatus)
     break(kind=bind, parent=vss1, child=n1),
     break(kind=bind, parent=vss1, child=n2),
     create(kind=vss, id=vss2),
     set(name=configStatus, owner=vss2, value=greeen),
     establish(kind=bind, parent=vss2, child=n1),
     establish(kind=bind, parent=vss2, child=n2),
]
```

For different types of objects, different actions can be taken:

* Resource objects could be created or removed.
* Attributes can be set and unset.
* Associations can be established and broke.

### 4. Resolve action sequence

Every action implicitly declares requirements about states. Only when states meet the requirements, actions can then be applied.

Here are the rules for such check.

| action | requirments |
| ---- | ---- |
| create | no resoure with the same id exists|
| remove | 1. specified resource object exists; <br> 2. All associations removed |
| set | owner resource exists |
| unset | 1. owner resource exists; <br> 2. attribute exists |
| establish | 1. the parent resource exists; <br> 2. the child resource exists; <br> 3. no association between the child and other resources |
| break | defined association exists |

In the example, for the initial state, it seems the following actions could be applied:

```javascript
[
     remove(kind=vss, id=vss1),
     unset(owner=vss1, name=configStatus)
     break(kind=bind, parent=vss1, child=n1),
     break(kind=bind, parent=vss1, child=n2),
     create(kind=vss, id=vss2),
]
```

Others can't be applied since `vss2` doesn't exist and both `n1` and `n2` are still assoicated with `vss1`.

If we consider the constraint: `network stay connected`, then we can't remove vss1.  For the same reason, we can't remove all the two nic from vss1 for now.

So next state(`state1`) could be:

```javascript
// unset(owner=vss1, name=configStatus)
// break(kind=bind, parent=vss1, child=n1),
// create(kind=vss, id=vss2),
{
    "vss":
    [
        {
            "key": "vss1",
            "nic":
            [
                "n2"
            ],
        },
        {
            "key": "vss2",
        }
    ]
}
```

Other actions can then be applied to `state1` and we get `state2`:

```javascript
// break(kind=bind, parent=vss1, child=n2),
// set(owner=vss2, name=configStatus)
// establish(kind=bind, parent=vss2, child=n1),
{
    "vss":
    [
        {
            "key": "vss1",
        },
        {
            "key": "vss2",
            "nic":
            [
                "n1"
            ],
            "configStatus": "green"

        }
    ]
}
```

After applying remained actions to `state2`, we reach the expcted final state.

## Model

```javascript


                                    ┌─────────────────┐
                                    │      Change     │
                                    └────────┬────────┘
                                             │                               ┌─────────────┐
                                             │          ┌────────────────────┤ Association │
                                             │          │                    └──────┬──────┘
                                             │          │                           │
                                             │          │                           │
                                        ┌────┴──────────▼──┐                        │
 ┌───────────────┐                 0..n │                  │               ┌────────┴────────┐
 │     state     ├──────────────────────┤    Object        ◄───────────────┤     Resource    │
 └───────────────┘                      │                  │               └────────┬────────┘
                                        └────┬──────────▲──┘                        │
                                             │          │                           │
                                             │          │                           │
                                             │          │                           │
                                             │          │                           │ 1..n
                                             │          │                      ┌────┴──────┐
                                             │          └──────────────────────┤ Attribute │
                                             │                                 └───────────┘
                                             │
                                             │
                                      ┌──────┴─────────┐
                                      │   Constraint   │
                                      └────────────────┘

```