# Attribute
Allgemeine Beschreibung der Selmo-Attribute, die für die Entwicklung der Funktion erforderlich sind.

## Deklaration
Die Attributte werden ausschlißlich in der Deklaration als Kommentar übergeben.
```cpp
///	[PERSISTENT(false)]
```

<details>
<summary> Code </summary>
	
```cpp
/// Powered by OSCAT www.oscat.de
/// Version 3.33
/// Modified by Selmo Technology
/// A PID controller with dynamic anti-wind up and manual control
///
/// version 1.2	
/// programmer	og         
/// tested by	hm   
/// [GROUP(Control engineering)] 
FUNCTION_BLOCK FB_CtrlPid
VAR_IN_OUT
	///	[PERSISTENT(false)]
	///	[HARDWARE(out)]
	///	[DESCRIPTION(Controller output)]	 
	///	
	Out_Y: REAL;
	///	[PERSISTENT(false)]
	///	[HARDWARE(in)]
	///	[DESCRIPTION(Actual Value)]
	///	
	In_ActValue: REAL;
END_VAR
VAR_INPUT
	///		 
	///	[PARAMETER(false)] 
	///	[ZONETYPE(inout)]
	///	[ZONENAME(Controller on)] 
	///	[ZONEGROUPNAME()]      
	///	[HMIBUTTON(true)] 
	///	[HMIBUTTONTEXT(Controller on)]  
	///	[HMIDISPLAYTEXT(Controller on)]
	///	[OUTPUTDESCRIPTION(Controller on)]
	///	[HARDWAREOUTPUT(false)] 
	///	[OUTPUTMODE(digital)]
	/// 	[RELATED_PARAMETERS(SetPoint,Suppression,OutputOffset,ManualInputValue,P_KP,I_TN,D_TV,LL,LH,Diff)]
	///	[ANALOGPARAMETER()] 
	///	[ANALOGVALUE()] 
	///	[PAIRCHECK(true)] 
	///	[PAIRCHECKGROUP(1)]
	///	
	ControllerOn: BOOL;
	///		 
	///	[PARAMETER(false)] 
	///	[ZONETYPE(inout)]
	///	[ZONENAME(Controller off)] 
	///	[ZONEGROUPNAME()]      
	///	[HMIBUTTON(true)] 
	///	[HMIBUTTONTEXT(Controller off)]  
	///	[HMIDISPLAYTEXT(Controller off)]
	///	[OUTPUTDESCRIPTION(Controller off)]
	///	[HARDWAREOUTPUT(false)] 
	///	[OUTPUTMODE(digital)]
	///	[RELATED_PARAMETERS()]
	///	[ANALOGPARAMETER()] 
	///	[ANALOGVALUE()] 
	///	[PAIRCHECK(true)] 
	///	[PAIRCHECKGROUP(1)]
	///	
	ControllerOff: BOOL;
	///sequence interface "read only"
	///
	///	[INTERFACE(stSequenceInterface)]
	///	
	stSeqIf: stSequenceInterface;
END_VAR
VAR_OUTPUT
	///	 			  
	///	[CMZ(false)] 
	///	[PARAMETER(false)] 
	///	[ZONETYPE(inout)] 
	///	[ZONENAME(Controller on)] 
	///	[ZONEGROUPNAME()] 
	///  	[CLONE2INVERTED(false)]
	///	[HMIDISPLAYTEXT(Controller is on)] 
	///	[HARDWAREINPUT(false)] 	
	///	[INPUTDESCRIPTION(Controller is on)] 
	///	[INPUTINVERTED(false)] 
	///	[INPUTDELAY(0)] 
	///	[INPUTMODE(digital)] 
	///	[RELATED_PARAMETERS()]
	///	[ANALOGPARAMETER()] 
	///	[ANALOGFUNCTION()] 
	///	[ANALOGVALUE()] 
	///	
	ControllerIsOn: BOOL;
	///	 			  
	///	[CMZ(false)] 
	///	[PARAMETER(false)] 
	///	[ZONETYPE(inout)] 
	///	[ZONENAME(Controller off)] 
	///	[ZONEGROUPNAME()] 
	///  	[CLONE2INVERTED(false)]
	///	[HMIDISPLAYTEXT(Controller is off)] 
	///	[HARDWAREINPUT(false)] 	
	///	[INPUTDESCRIPTION(Controller is off)] 
	///	[INPUTINVERTED(false)] 
	///	[INPUTDELAY(0)] 
	///	[INPUTMODE(digital)] 
	///	[RELATED_PARAMETERS()]
	///	[ANALOGPARAMETER()] 
	///	[ANALOGFUNCTION()] 
	///	[ANALOGVALUE()] 
	///	
	ControllerIsOff: BOOL;
	///	 			  
	///	[CMZ(false)] 
	///	[PARAMETER(false)] 
	///	[ZONETYPE(in)] 
	///	[ZONENAME(Controller Limit)] 
	///	[ZONEGROUPNAME()] 
	///  	[CLONE2INVERTED(false)]
	///	[HMIDISPLAYTEXT(Controller Limit detection)] 
	///	[HARDWAREINPUT(false)] 	
	///	[INPUTDESCRIPTION(Controller Limit detection)] 
	///	[INPUTINVERTED(false)] 
	///	[INPUTDELAY(0)] 
	///	[INPUTMODE(digital)] 
	///	[RELATED_PARAMETERS()]
	///	[ANALOGPARAMETER()] 
	///	[ANALOGFUNCTION()] 
	///	[ANALOGVALUE()] 
	///	
	LimitDetection: BOOL;
END_VAR
VAR_INPUT
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(output)]
	///	[HMIDISPLAYTEXT(Actual Value)] 
	///	[INITIALVALUE()]	 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	ActValue: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(Set Point)] 
	///	[UNIT()] 
	///	[INITIALVALUE()]	 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	SetPoint: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(Suppression)]
	///	[INITIALVALUE()] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	Suppression: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(Offset)]
	///	[INITIALVALUE()] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	OutputOffset: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(Input value for manual operation)] 
	///	[INITIALVALUE()]	 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	ManualInputValue: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(P_KP gain)]
	///	[INITIALVALUE(1)] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	P_KP: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(I_TN integral time)]
	///	[INITIALVALUE(1)] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	I_TN: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(D_TV derivative time)]
	///	[INITIALVALUE(1)] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	D_TV: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(lower output limit)]
	///	[INITIALVALUE(-100)] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	LL: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(input)]
	///	[HMIDISPLAYTEXT(higher output limit)]
	///	[INITIALVALUE(100)] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	LH: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(output)]
	///	[HMIDISPLAYTEXT(output)]
	///	[INITIALVALUE()] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	Y: REFERENCE TO REAL;
	///		 	 
	///	[PARAMETER(true)] 
	///	[TYPE(output)]
	///	[HMIDISPLAYTEXT(deviation)] 
	///	[UNIT()] 
	///	[LIMITMIN()] 
	///	[LIMITMAX()] 
	///	[DECIMALDIGITS(4)] 
	///	[SECTION()]
	///	[DISABLEAUTO(false)] 
	///	[BUTTONMODE()] 
	///	
	{attribute 'input_constant' := ''}
	Diff: REFERENCE TO REAL;
END_VAR
VAR
	fbCTRL_PID: CTRL_PID;
	F_TRIGAuto: F_TRIG;
	/// One cycle initialization
	xInit: BOOL;
END_VAR
```
</details>

## Übersicht der Attribute
Insgesamt sind Attribute in der Programmierung grundlegend für die Organisation und Verarbeitung von Daten und Informationen in Programmen und Anwendungen.
Bei Selmo werden folgende Attribute verwendet:

- [ANALOGPARAMETER](##analogparameter)
- [ANALOGFUNCTION](###analogfunction)
- [ANALOGVALUE](###analogvalue)
- [AUTORESET](###autoreset)
- [BUTTONMODE](###buttonmode) 
- [CLONE2INVERTED](###clone2inverted)
- [CMZ]
[DECIMALDIGITS] 
[DECLARATIONASINPUT
[DISABLEAUTO
[ERRORDELAY
[GHOSTMODE
[GHOSTMODEDELAYDELAY
[HARDWARE
[HARDWAREINPUT
[HARDWAREOUTPUT
[HMIBUTTON
[HMIBUTTONTEXT
[HMIDISPLAYTEXT
[INPUTDELAY
[INPUTDESCRIPTION
[INPUTINVERTED
[INPUTMODE
[INVERTED
[KEEPOUTPUTALIVE
[LIMITMAX
[LIMITMIN 
[OUTPUTDESCRIPTION
[OUTPUTGROUP
[OUTPUTMODE
[PAIRCHECK
[PAIRCHECKGROUP
[PARAMETER
[PERSISTENT
[RELATED_PARAMETERS
[SECTION
[TYPE
[UNIT 
[ZONEGROUPNAME
[ZONENAME
[ZONETYPE

## ANALOGPARAMETER

### ANALOGPARAMETER

### ANALOGFUNCTION

### ANALOGVALUE

### AUTORESET

### BUTTONMODE 

### CLONE2INVERTED

### CMZ

### DECIMALDIGITS 

### DECLARATIONASINPUT

### DISABLEAUTO

### ERRORDELAY

### GHOSTMODE

### GHOSTMODEDELAYDELAY

### HARDWARE

### HARDWAREINPUT

### HARDWAREOUTPUT

### HMIBUTTON

### HMIBUTTONTEXT

### HMIDISPLAYTEXT

### INPUTDELAY

### INPUTDESCRIPTION

### INPUTINVERTED

### INPUTMODE

### INVERTED

### KEEPOUTPUTALIVE

### LIMITMAX

### LIMITMIN 

### OUTPUTDESCRIPTION

### OUTPUTGROUP

### OUTPUTMODE

### PAIRCHECK

### PAIRCHECKGROUP

### PARAMETER

### PERSISTENT

### RELATED_PARAMETERS

### SECTION

### TYPE

### UNIT 

### ZONEGROUPNAME

### ZONENAME

### ZONETYPE

