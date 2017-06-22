# SOFSEM 2018 - SUPLEMENTARY Material

In this page, we present supporting material of the paper entitled "UML2PROV: Automating Provenance Capture in Software Engineering" submitted to the 44th International Conference on Current Trends in Theory and Practice of Computer Science. January 29 - February 2, 2018 | Krems an der Donau, Austria.

* [OCL Constraints](https://github.com/uml2prov/uml2prov.github.io/blob/master/README.md#OCL-Constraints)
* [Translation rules](https://github.com/uml2prov/uml2prov.github.io/blob/master/README.md#translation-rules)
* [Evaluation dataset](https://github.com/uml2prov/uml2prov.github.io/blob/master/README.md#evaluation-dataset)


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




## Translation rules

Here we present a complete description of the rules used to mapping UML Sequence Diagrams (SqD) and UML State Diagrams (SMD) to provenance templates. Such rules have been divided into those that deal with SqD, and those that tackle SMD. 

### From Sequence Diagrams to templates

* SeqR1. The _lifeline_ representing the sender of a _message_ is mapped to a PROV __agent__, whose type is given by the name of its class. 

* SeqR2. Each _message_ is mapped to a PROV __activity__, whose type is given by the message's name, and a start time, given by the _message_ invocation time. In addition, the PROV __activity__ representing the _message_ is related to the PROV __agent__ representing the sernder _lifeline_ (given by SeqR1) through the PROV relationship __prov:wasAssociatedWith__.

* SeqR3. Each _input argument_ within an _asynchronous_ or _synchronous_ _message_ is mapped to a PROV entity identified by the variable var:inputN, where N refers to the argument's position, and a PROV attribute __prov:value__. Since _input arguments_ are "used" by _message_ to perform their particular operations, we utilize the relationship __prov:used__ to relate the __activity__, created for the _message_ (by SeqR2), with each __entity__, created for each _input argument_. Besides, the relationship used has a PROV attribute __prov:role__ with the name of the _input argument_. 

* SeqR4. Each _output argument_ within a _reply_ _message_ is mapped to a PROV __entity__ identified by the variable var:outputN, where N refers to the argument's position, and the attribute __prov:value__. Since it could be said that each _output argument_ is "generated" by a reply _message_, we use the PROV association __prov:wasGeneratedBy__ to relate the __activity__, created for the _message_ (by SeqR2), with the __entity__, created for each _output argument_. Furthermore, __prov:wasGeneratedBy__ has a PROV attribute __prov:role__ with the name of the _output argument_.

* SeqR5. In PROV two relationships with the form (B, __prov:used__, A) and (C, __prov:wasGeneratedBy__, B) (see SeqR3 and SeqR4, respectively), have to be enriched with a new direct relationship between (C, __prov:wasDerivedFrom__, A) to express the dependency of C on A. This structure refers to a provenance construction called _Use-generate-derive triangle_, which encloses the three elements involved.

### From State Machine Diagrams to templates

* StR1. The _object_ of a _transition_ is mapped to a PROV __agent__, with the type attribute given by the name of the _object_'s class. Furthermore, the _object_'s _state machine_ is represented as a PROV __entity__ (we note that it has a similar translation to a SqD's _lifeline_). More specifically, we create an __entity__ related to the _object_'s _state machine_ to have an element representing the umbrella for the _object_'s concrete _states_. Both the PROV __agent__ representing the _object_ and the PROV __entity__ representing the _object_'s _state machine_ are related by means of __prov:wasAttributedTo__.

* StR2. The _event_ in a _transition_ is mapped to a PROV __activity__, whose type is given by the _transition_ name, and start time, given by the _transition_ invocation time. 

* Str3. Each _state_ (_simple_ or _composite_) included in the SMD is mapped to a PROV __entity__. If the _state_ constitutes the _source_ of the _transition_, the new PROV __entity__ is identified by var:source, and is related to the PROV __activity__ which represents the _transition_ through the relationship __prov:used__. In addition, in order to represent the fact that the triggered of a _transition_ changes the _object_'s active _state_ from a _source_ _state_ to a _target_ _state_, the _source_ _state_ PROV __entity__ is related to the PROV __activity__ translation of the _transition_ by means of the __prov:wasInvalidatedBy__ relationship. In contrast, if the _state_ corresponds to the target of the _transition_, the new PROV __entity__ is identified by var:target, and is related to the PROV __activity__ which represents the _transition_ by means of the __prov:wasGeneratedBy__ relationship. In both cases, the PROV __entity__ is complemented by the attributes __prov:type__ and __u2p:state__. While __prov:type__ is given by the name of the _object_ modelled by the SMD, __u2p:state__ is given by the name of the translated _state_. 

* Str4. As in SeqR5, we can identify the _Use-generate-derive triangle_ among the PROV __entity__ representing the _source_ _state_, the PROV __activity__ representing the _transition_ and the PROV __entity__ representing the _target_ _state_. Taking this into account, we define a direct relationship between both the source PROV __entity__ and the target PROV __entity__ by means of the __prov:wasDerivedFrom__ relationship, representing the fact that the _target_ _state_ is a consequence of the triggered of the _transition_ from the _source_ _state_. 

* Str5. The relationship between the _object_ whose behaviour is modelled and its states depend on whether the _transition_ is enclosed in a _composite_ _state_ or not. If the _transition_ is enclosed, in addition to create a PROV __entity__ representing the _composite_ _state_ (StR3), we register such a _state_ as belonging to the _object_'s _state machine_ by means of the __prov:specializationOf__ relationship. Additionally, each PROV __entity__ representing an enclosed state is directly related with the PROV __entity__ representing the _composite_ _state_ through __prov:hadMember__. It allows us to assert that each substate "is a member of" the _composite_ _state_. In contrast, if the _transition_ is not enclosed, we register each _state_ which composes the _transition_ as belonging to the _object_'s _state machine_ by means of the __prov:specializationOf__ relationship. 


## Evaluation dataset

In this section we provide the dataset which includes the relevant documents related to the case studies used in "Analysis and Discussion" section. 

* [Seminar](https://github.com/uml2prov/uml2prov.github.io/tree/master/Seminar)
* [Water](https://github.com/uml2prov/uml2prov.github.io/tree/master/Water)
* [Model View Controller (MVC)](https://github.com/uml2prov/esec-fse/tree/master/MVC) 
* [PhoneCall](https://github.com/uml2prov/uml2prov.github.io/tree/master/PhoneCall)
* [Elevator](https://github.com/uml2prov/uml2prov.github.io/tree/master/Elevator)










