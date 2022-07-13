# UnitTest_RuleSet
IRIS UnitTest TestCase Template. UnitTest for HL7 Routing Rules in IRIS Integration.

Code consists of:
* UnitTest.RuleSet.HL7 - Subclass this to generate new unit tests for your HL7 routing rules

Also included to demonstrate implementation:
* UnitTest.RuleSet.Example - An example subclass to demonstrate how to create routing unit tests
* UnitTest.RuleSet.TestHL7 - An example production where 
* UnitTest.RuleSet.TestHL7.RoutingRule - An example routing rules to be tested

To run all UnitTest on a new UnitTest class manually:
```
Do ##class(UnitTest.RuleSet.Example).Debug()
```

# Environment configuration
Leveraging the built-in TestCase Runner we require that the global "UnitTestRoot" points to a physical directory for example "c:\temp"
```
Set ^UnitTestRoot="c:\temp"
```
TestCases contain a suite name parameter. So if this is called "RuleSetHL7" you also need to create a subdirectory with this name
```
mkdir c:\temp\RuleSetHL7
```

# How to create a new HL7 routing UnitTest

Define a class overriding <em>UnitTest.RuleSet.HL7</em> :
1. Parameter TestSuite - To a suitable name if required
2. Parameter HL7Schema - The HL7 Schema of your message. Useful to augment per-Test message content, before sending to be evaluated for routing.
3. Parameter SourceBusinessServiceName - The Name of your HL7 Business Service in your production. Not to update the pool size to 2 for testing.
4. Parameter TargetConfigName = The name of your Business Process that is an instance of an HL7 Routing Engine
5. Create new Methods with names prefixed "Test". These will invoke the method "SendMessage" with critera about the message routing result expected.
6. XData SourceMessage - Replace with base HL7 messages suitable to your business need. Note the HL7Schema should match to effect per-test updates.

```
/// d ##class(UnitTest.RuleSet.Example).Debug()
Class UnitTest.RuleSet.Example Extends UnitTest.RuleSet.HL7
{

Parameter TestSuite = "RuleSetHL7";

/// Override for different schema
Parameter HL7Schema = "2.5.1:OUL_R22";

/// Override with name of existing service on production
Parameter SourceBusinessServiceName = "From_SystemZ";

/// Override with name of existing business process routing engine on production
Parameter TargetConfigName = "MsgRouter";

Method TestMessageA()
{
	#dim message as EnsLib.HL7.Message
	set message=..HL7Message.%ConstructClone(1)
	do message.PokeDocType(message.DocType)
	do message.SetValueAt("SYSA","MSH:3.1")
	set expectSuccess=1
	set expectReturn="send:To_SystemA:"
	set expectReason="rule#1"
	do ..SendMessage(message,"TestMessageA", expectSuccess, expectReturn, expectReason)
}

XData SourceMessage
{
<test><![CDATA[
MSH|^~\&|FRM|FRM5T|SYSA|SYSA|202207121010||OUL^R22^OUL_R22|1234567|P|2.5|||||GBR||EN
PID|||123456789^^^NHS^NH~12345^^^774^PI||SMITH^BOB^^^||19830501|M||||||||||||||||||||||N
PV1||N|ANT5TREE^^^^^^^^MH Anticoagulant Clinic,||||C123546^Bowden^S M^^^Dr|||101|ANT5T^^^^^^^^Harbor|||||||||||||||||||||||||||||||||20220712101010
SPM|1|^62000001||03^PLA-CIT^DDN|||||||||||||20220712101010|20220712101010
OBR|1||120000000001^DDN|^International normalised ratio (INR) dose^RC^B0281^INRD^TCL||20220712101010||||||||||C123546^Bowden^S M^^^Dr|||||1|20220712101010||B|C|||||||tim223|||||||||||||||HNM^Haematology(Multiple T/S)^^AUTH
ORC|SC||120000000001^DDN||||||20220712101010|||C123546^Bowden^S M^^^Dr|||||||||Mariden University Health Board^^ANT5|||||||N
OBX|1|SN|42QE.^International normalised ratio^RC^B0700^INR^TCL|1|^1.6|ratio^^TCL||N|||C|||20150519122300||tim223
OBX|2|TX|^Warfarin Dose^RC^B0850^DOSE^TCL|2||||N|||C|||20220712101010||tim223
NTE|1|L|INR: 1.6 Target 3.0 - 4.0  L
NTE|2|L|Full Dosing Instruction:
NTE|3|L|7 mg DAILY
NTE|4|L|Next Test: 16/07/2022
]]></test>
}

```

