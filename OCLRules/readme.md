
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
