
## OCL Constraints

These constraints (from OCL1 to OCL5) mainly impose interconnections
among (OCL1 and OCL2) senders/receivers in SqD and objects
modelled by SMD, (OCL3 and OCL4) incoming/outgoing messages in
SqD with events/actions in SMD, and (OCL5) incoming messages in
SqD with methods in objects modelled by SMD. 

**(OCL1-2) Each sender of a message in an interaction of a SqD must be an object modelled by a SMD. The same constraint is defined for a receiver, changing sender by receiver.**

```
context: Interaction
inv: self.message.sender.base.behavior -> notEmpty ()
```

**(OCL3) Incoming messages to an object within a SqD are events in a SMD.**
```
context: Message
inv: self.receiver.base.behavior.region.transition.trigger-> exists(e| e.name=self.name)
```

**(OCL4) Outgoing messages of an object within a SqD are actions in a SMD.**
```
context: Message
inv: self.sender.base.behavior.region.transition.effect-> exists(e| e.name=self.name )

```

**(OCL5) Incoming messages of an object (receiver) within SqD must be object's methods.**

```
context: Message
inv: let rec:ClassifiedRole = self.receiver in
          let ops:Operation = rec.base.ownedOperation in
          ops -> exists(oper| oper.name = self.name)
```


**(OCL6) If a message has a signature, it must be an operation.**

Regarding SqD diagrams, we have defined the constraint labelled
OCL6 to make sure that when a message includes a signature,
such signature must correspond to an operation (instead of
a signal). The approach presented in this paper considers operation
messages, but it could be easily extended to include signals.

```
context: Message
inv: self.signature ->notEmpty ()
      implies self.signature.oclIsKindOf (Operation)
```


**(OCL7) No submachine states are considered as valid states in a SMD.**

Since submachine states are semantically equivalent
to composite states [26], we have not considered these kind of
states in this paper. 
This aspect is modelled by means of constraint OCL7.

```
context: State
inv: not self.isSubmachineState()
```



**(OCL8-16) Only initial pseudostates are considered as valid pseudostates in a SMD. Parameter \<kind> must be substituted by: deep/ shallowHistory, join, fork, junction, choice, entry/exitPoint and terminate.**

Constraints from OCL8 to OCL16
ensure that the remainder kinds of pseudostate elements can not
be source/target vertices. Such constraints share a same pattern, where the parameter <kind> must be
substituted by each pseudostate not considered by UML2PROV.

```
context: Transition
inv: not self.source.oclAstype (Pseudostate ).kind = PseudostateKind ::<kind> and
     not self.target.oclAstype (Pseudostate ).kind = PseudostateKind ::<kind>


```
