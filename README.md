# Attribute
Allgemeine Beschreibung der Selmo-Attribute, die für die Entwicklung der Funktion erforderlich sind.

## Deklaration
Die Attributte werden ausschlißlich in der Deklaration als Kommentar übergeben.

### Header
Hier wird die Funktion so detailliert wie möglich beschrieben.
```cpp
/// Powered by OSCAT www.oscat.de
/// Version 3.33
/// Modified by Selmo Technology
/// A PID controller with dynamic anti-wind up and manual control
///
/// version 1.3	
/// programmer	og         
/// tested by	hm   
/// [GROUP(Control engineering)] 
```

### VAR_IN_OUT
Dies ist vorgesehen, um reale Hardware zu steuern.

#### Input 
```cpp  
///	[PERSISTENT(false)]
///	[HARDWARE(in)]
///	[DESCRIPTION(Actual Value)]
///	
In_ActValue: REAL;
```       
#### Output
```cpp  
///	[PERSISTENT(false)]
///	[HARDWARE(out)]
///	[DESCRIPTION(Controller output)]	 
///	
Out_Y: REAL;
```

### VAR_INPUT
VAR_INPUT wird verwendet, um die Ausgänge der Zone zu verbinden, CMZs zu generieren, zugehörige Parameter zu erstellen und Schnittstellen der Selmowelt abzufragen.  

#### Zone InOut
```cpp
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
///     [RELATED_PARAMETERS(SetPoint,Suppression,OutputOffset,ManualInputValue,P_KP,I_TN,D_TV,LL,LH,Diff)]
///	[ANALOGPARAMETER()] 
///	[ANALOGVALUE()] 
///	[PAIRCHECK(true)] 
///	[PAIRCHECKGROUP(1)]
///	
ControllerOn: BOOL;
```

#### Zone Out
```cpp       
///	[PARAMETER(false)] 
///	[ZONETYPE(out)]
///	[ZONENAME(Set)] 
///	[ZONEGROUPNAME(Set)]   
///	[HMIBUTTON(true)] 
///	[HMIBUTTONTEXT(Set)]  
///	[HMIDISPLAYTEXT(Set)]
///	[OUTPUTDESCRIPTION(Set)]
///	[HARDWAREOUTPUT(false)] 
///	[OUTPUTMODE(digital)] 
///	[ANALOGPARAMETER()] 
///	[ANALOGVALUE()] 
///	[PAIRCHECK(false)] 
///	[PAIRCHECKGROUP()]
///   
Set : BOOL;
```

#### CMZ
```cpp  
///	[CMZ(true)] 
///	[PARAMETER(false)] 
///	[HMIDISPLAYTEXT(Timeout communication)]  
///	[INVERTED(false)]
///	[DECLARATIONASINPUT(false)] 
///	[AUTORESET(false)]   
///	[ERRORDELAY(0)] 
///  
TimeoutComm: BOOL;
```

#### Parameter
```cpp
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
```

#### Schnittstelle
```cpp
///sequence interface "read only"
///
///	[INTERFACE(stSequenceInterface)]
///	
stSeqIf: stSequenceInterface;
```

### VAR_OUTPUT
VAR_OUTPUT wird verwendet, um die Eingänge der Zone zu verbinden.  
#### Zone InOut
```cpp
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
```

#### Zone In
```cpp
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
```

### VAR
```cpp
fbCTRL_PID: CTRL_PID;
F_TRIGAuto: F_TRIG;
/// One cycle initialization
xInit: BOOL;
```

### Code

#### Initialisierung
Der Codeabschnitt `//initialize procedure` enthält, wie der Name schon sagt, Initialisierungswerte und ist ein fester Bestandteil jeder Selmo-Funktion.
```cpp
//initialize procedure
IF NOT xInit THEN 
	ControllerIsOn := FALSE; 
	ControllerIsOff := TRUE; 

	xInit:=TRUE;
END_IF
```
#### Signaldeaktivierung 
Der Codeabschnitt `//Signals deactivate on falling edge of the automatic release.` beinhaltet, wie der Name bereits impliziert, Signale, die zwingend deaktiviert bzw. aktiviert werden müssen und stellt einen festen Bestandteil jeder Selmo-Funktion dar.
```cpp
//Signals deactivate on falling edge of the automatic release.
F_TRIGAuto(CLK:=stSeqIf.xSeqAutomaticReleased, Q=> );

IF F_TRIGAuto.Q OR NOT stSeqIf.xNoCMZFault THEN
	ControllerIsOn := FALSE; 
	ControllerIsOff := TRUE; 
ELSE
	ControllerIsOn S= ControllerOn; 
	ControllerIsOn R= ControllerOff; 
	ControllerIsOff S= ControllerOff; 
	ControllerIsOff R= ControllerOn;  
END_IF
```
#### Funktion
```cpp
ActValue := In_ActValue;

fbCTRL_PID(
	ACT:=ActValue , 
	SET:=SetPoint , 
	SUP:=Suppression , 
	OFS:=OutputOffset , 
	M_I:=ManualInputValue , 
	MAN:=ControllerIsOff , 
	RST:=ControllerIsOff , 
	KP:=P_KP, 
	TN:=I_TN , 
	TV:=D_TV , 
	LL:=LL , 
	LH:=LH , 
	//Y=>Y , 
	DIFF=>Diff , 
	LIM=>LimitDetection );
		
IF INV THEN 
	Out_Y := fbCTRL_PID.Y*-1;
ELSE
	Out_Y := fbCTRL_PID.Y;
END_IF

Y:=Out_Y;
Out_Y_Int := REAL_TO_INT(Out_Y);
```

<details>
<summary> Deklarations Beispiel </summary>
	
</details>

## Übersicht der Attribute
Insgesamt sind Attribute in der Programmierung grundlegend für die Organisation und Verarbeitung von Daten und Informationen in Programmen und Anwendungen.
Bei Selmo werden folgende Attribute verwendet:

- [ANALOGPARAMETER](#analogparameter)
- [ANALOGFUNCTION](###analogfunction)
- [ANALOGVALUE](###analogvalue)
- [AUTORESET](###autoreset)
- [BUTTONMODE](###buttonmode) 
- [CLONE2INVERTED](###clone2inverted)
- [CMZ](###cmz)
- [DECIMALDIGITS](###decimaldigits) 
- [DECLARATIONASINPUT](###declarationasinput)
- [DISABLEAUTO](###disableauto)
- [ERRORDELAY](###errordelay)
- [GHOSTMODE](###ghostmode)
- [GHOSTMODEDELAYDELAY](###ghostmodedelaydelay)
- [HARDWARE](###hardware)
- [HARDWAREINPUT](###hardwareinput)
- [HARDWAREOUTPUT](###hardwareoutput)
- [HMIBUTTON](###hmibutton)
- [HMIBUTTONTEXT](###hmibuttontext)
- [HMIDISPLAYTEXT](###hmidisplaytext)
- [INPUTDELAY](###inputdelay)
- [INPUTDESCRIPTION](###inputdescription)
- [INPUTINVERTED](###inputinverted)
- [INPUTMODE](###inputmode)
- [INVERTED](###inverted)
- [KEEPOUTPUTALIVE](###keepoutputalive)
- [LIMITMAX](###limitmax)
- [LIMITMIN](###limitmin)
- [OUTPUTDESCRIPTION](###outputdescription)
- [OUTPUTGROUP](###outputgroup)
- [OUTPUTMODE](###outputmode)
- [PAIRCHECK](###paircheck)
- [PAIRCHECKGROUP](###paircheckgroup)
- [PARAMETER](###parameter)
- [PERSISTENT](###persistent)
- [RELATED_PARAMETERS](###related_parameters)
- [SECTION](###section)
- [TYPE](###type)
- [UNIT](###unit)
- [ZONEGROUPNAME](###zonegroupname)
- [ZONENAME](###zonename)
- [ZONETYPE](###zonetype)

# ANALOGPARAMETER

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

