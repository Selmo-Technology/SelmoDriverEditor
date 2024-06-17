# Selmo Funktion
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
- [ANALOGFUNCTION](#analogfunction)
- [ANALOGVALUE](#analogvalue)
- [AUTORESET](#autoreset)
- [BUTTONMODE](#buttonmode) 
- [CLONE2INVERTED](#clone2inverted)
- [CMZ](#cmz)
- [DECIMALDIGITS](#decimaldigits) 
- [DECLARATIONASINPUT](#declarationasinput)
- [DISABLEAUTO](#disableauto)
- [ERRORDELAY](#errordelay)
- [GHOSTMODE](#ghostmode)
- [GHOSTMODEDELAYDELAY](#ghostmodedelaydelay)
- [HARDWARE](#hardware)
- [HARDWAREINPUT](#hardwareinput)
- [HARDWAREOUTPUT](#hardwareoutput)
- [HMIBUTTON](#hmibutton)
- [HMIBUTTONTEXT](#hmibuttontext)
- [HMIDISPLAYTEXT](#hmidisplaytext)
- [INPUTDELAY](#inputdelay)
- [INPUTDESCRIPTION](#inputdescription)
- [INPUTINVERTED](#inputinverted)
- [INPUTMODE](#inputmode)
- [INVERTED](#inverted)
- [KEEPOUTPUTALIVE](#keepoutputalive)
- [LIMITMAX](#limitmax)
- [LIMITMIN](#limitmin)
- [OUTPUTDESCRIPTION](#outputdescription)
- [OUTPUTGROUP](#outputgroup)
- [OUTPUTMODE](#outputmode)
- [PAIRCHECK](#paircheck)
- [PAIRCHECKGROUP](#paircheckgroup)
- [PARAMETER](#parameter)
- [PERSISTENT](#persistent)
- [RELATED_PARAMETERS](#related_parameters)
- [SECTION](#section)
- [TYPE](#type)
- [UNIT](#unit)
- [ZONEGROUPNAME](#zonegroupname)
- [ZONENAME](#zonename)
- [ZONETYPE](#zonetype)

### ANALOGPARAMETER
Deklarationsbereich
[VAR_OUTPUT](#var_output)

Objektbereich 
[Zone In](#zone-in), [Zone InOut](#zone-inout) 

Syntax
```cpp
[ANALOGFUNCTION(Equals)], [ANALOGFUNCTION(GreaterThan)], [ANALOGFUNCTION(LessThan)], [ANALOGFUNCTION(GreaterEquals)], [ANALOGFUNCTION(LessEquals)]
```

Beschreibung
Die in eckigen Klammern angegebenen Ausdrücke, wie "Equals" (Gleich), "GreaterThan" (Größer als), "LessThan" (Kleiner als), "GreaterEquals" (Größer oder gleich) und "LessEquals" (Kleiner oder gleich), repräsentieren verschiedene Vergleichsoperationen oder Bedingungen, die in der "ANALOGFUNCTION" verwendet werden können.
Equals: Diese Funktion dient dazu, zu überprüfen, ob zwei analoge Werte gleich sind. Sie könnte beispielsweise verwendet werden, um festzustellen, ob ein bestimmtes analoges Signal einem anderen entspricht.
GreaterThan: Diese Funktion wird verwendet, um zu überprüfen, ob ein analoger Wert größer ist als ein anderer. Dies kann nützlich sein, um Bedingungen zu definieren, die erfüllt werden müssen, wenn ein Wert eine bestimmte Schwelle überschreitet.
LessThan: Im Gegensatz zur vorherigen Funktion überprüft diese, ob ein analoger Wert kleiner ist als ein anderer. Das kann in Situationen hilfreich sein, in denen die Größe eines Wertes von Bedeutung ist.
GreaterEquals: Diese Funktion dient dazu, festzustellen, ob ein Wert gleich oder größer als ein anderer ist. Das ist nützlich, wenn Sie eine Aktion auslösen möchten, wenn ein Wert einen bestimmten Schwellenwert erreicht oder überschreitet.
LessEquals: Hierbei wird überprüft, ob ein Wert kleiner oder gleich einem anderen ist. Dies kann in Szenarien verwendet werden, in denen Sie Aktionen basierend auf einer bestimmten Grenze auslösen möchten.

### ANALOGFUNCTION
Deklarationsbereich
[VAR_INPUT](#var_input), [VAR_OUTPUT](#var_output)

Objektbereich
[Zone In](#zone-in), [Zone InOut](#zone-inout), [Zone Out](#zone-out)

Beschreibung
Dies repräsentiert einen Parameter, der dazu verwendet wird, einen bestimmten Wert oder eine Variable zu identifizieren, die in einem Vergleich oder einer Operation verwendet werden soll.

Syntax
```cpp
[ANALOGPARAMETER(ValueX1)]
```

### ANALOGVALUE

### AUTORESET

### BUTTONMODE 
![](data:image/png;base64,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)

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

