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
- [GHOSTMODEDELAY](#ghostmodedelay)
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
[Zone In](#zone-in) , [Zone InOut](#zone-inout) 

Syntax  
```cpp
[ANALOGFUNCTION(Equals)], [ANALOGFUNCTION(GreaterThan)], [ANALOGFUNCTION(LessThan)], [ANALOGFUNCTION(GreaterEquals)], [ANALOGFUNCTION(LessEquals)]
```

Beschreibung  
Die in eckigen Klammern angegebenen Ausdrücke, wie "Equals" (Gleich), "GreaterThan" (Größer als), "LessThan" (Kleiner als), "GreaterEquals" (Größer oder gleich) und "LessEquals" (Kleiner oder gleich), repräsentieren verschiedene Vergleichsoperationen oder Bedingungen, die in der "ANALOGFUNCTION" verwendet werden können.
- Equals: Diese Funktion dient dazu, zu überprüfen, ob zwei analoge Werte gleich sind. Sie könnte beispielsweise verwendet werden, um festzustellen, ob ein bestimmtes analoges Signal einem anderen entspricht.
- GreaterThan: Diese Funktion wird verwendet, um zu überprüfen, ob ein analoger Wert größer ist als ein anderer. Dies kann nützlich sein, um Bedingungen zu definieren, die erfüllt werden müssen, wenn ein Wert eine bestimmte Schwelle überschreitet.
- LessThan: Im Gegensatz zur vorherigen Funktion überprüft diese, ob ein analoger Wert kleiner ist als ein anderer. Das kann in Situationen hilfreich sein, in denen die Größe eines Wertes von Bedeutung ist.
- GreaterEquals: Diese Funktion dient dazu, festzustellen, ob ein Wert gleich oder größer als ein anderer ist. Das ist nützlich, wenn Sie eine Aktion auslösen möchten, wenn ein Wert einen bestimmten Schwellenwert erreicht oder überschreitet.
- LessEquals: Hierbei wird überprüft, ob ein Wert kleiner oder gleich einem anderen ist. Dies kann in Szenarien verwendet werden, in denen Sie Aktionen basierend auf einer bestimmten Grenze auslösen möchten.

### ANALOGFUNCTION
Deklarationsbereich  
[VAR_INPUT](#var_input), [VAR_OUTPUT](#var_output)

Objektbereich  
[Zone In](#zone-in), [Zone InOut](#zone-inout), [Zone Out](#zone-out)

Syntax  
```cpp
[ANALOGPARAMETER(ValueX1)]
```
Beschreibung  
Dies repräsentiert einen Parameter, der dazu verwendet wird, einen bestimmten Wert oder eine Variable zu identifizieren, die in einem Vergleich oder einer Operation verwendet werden soll.

### ANALOGVALUE
Deklarationsbereich  
[VAR_INPUT](#var_input), [VAR_OUTPUT](#var_output)

Objektbereich  
[Zone In](#zone-in), [Zone InOut](#zone-inout), [Zone Out](#zone-out)

Syntax  
```cpp
[ANALOGVALUE(100)]
```
Beschreibung  
Dies dient zur Festlegung eines konkreten analogen Werts in einem programmatischen Kontext.

### AUTORESET
Deklarationsbereich  
[VAR_OUTPUT](#var_output)

Objektbereich  
[CMZ](#cmz)

Syntax  
```cpp
[AUTORESET(false)], [AUTORESET(true)]
```
Beschreibung  
Ein Auto Reset im Fehlerfall bezeichnet eine Funktion, bei der ein Fehler automatisch zurückgesetzt wird, ohne dass manuell eingegriffen werden muss. Wenn ein Fehler auftritt, wird dieser automatisch erkannt und der Systemzustand wird auf den normalen Betriebszustand zurückgesetzt. Diese Funktion wird häufig in automatisierten Systemen verwendet, um sicherzustellen, dass der Betrieb fortgesetzt werden kann, ohne dass ein Bediener manuell eingreifen muss, um den Fehler zu beheben. Das Auto Reset im Fehlerfall ist besonders nützlich in kritischen Anwendungen, bei denen ein sofortiges Eingreifen notwendig ist, um Ausfallzeiten oder Schäden an der Ausrüstung zu minimieren. 

### BUTTONMODE 
Deklarationsbereich  
[VAR_INPUT](#var_input)

Objektbereich  
[PARAMETER](#parameter)

Syntax  
```cpp
[BUTTONMODE(On)], [BUTTONMODE(Off)], [BUTTONMODE(Switch)], [BUTTONMODE(Toggle)]
```
Beschreibung  
Boolesche Parameter können als Buttons angelegt werden. Es stehen verschiedene Modi zur Verfügung:

- On: Schaltet die Parametervariable auf "true" beim Betätigen des HMI-Buttons.
- Off: Schaltet die Parametervariable auf "false" beim Betätigen des HMI-Buttons.
- Switch: Schaltet die Parametervariable auf "true" beim Drücken und auf "false" beim Loslassen des HMI-Buttons.
- Toggle: Wechselt bei jedem Tastendruck den Zustand der Parametervariable zwischen "true" und "false".

Wenn der HMI-Button im grünen Zustand ist, entspricht dies dem Wert 'true' der Parametervariable. Wenn der Button im grauen Zustand ist, entspricht dies dem Wert 'false'.

![HMI](images/HMIParameterButtonMode.png)

### CLONE2INVERTED
Deklarationsbereich  
[VAR_OUTPUT](#var_output)

Objektbereich  
[Zone In](#zone-in), [Zone InOut](#zone-inout)

Syntax  
```cpp
[CLONE2INVERTED(false)], [CLONE2INVERTED(true)]
```
Beschreibung  
Um ein Signal vollständig abzusichern kann eine invertierte Zone-In eingefügt werden. Die invertierte Zone überwacht den sicheren Übergang des mit der Zone verknüpften Signals von true auf false. Beispielsweise werden Taster, Sensoren etc. damit überwacht. Dadurch kann eine Fehlbedienung verhindert werden.  Mithilfe des Buttons Clone to inverted wird eine invertierte Zone der ausgewählten Zone-In eingefügt.

![Studio](images/InvertierteZoneIn.png)

### CMZ
Deklarationsbereich  
[VAR_OUTPUT](#var_output)

Objektbereich  
[CMZ](#cmz)

Syntax  
```cpp
[CMZ(false)], [CMZ(true)]
```
Beschreibung  
Wird verwendet, um ein CMZ im CMZ-Bereich der Sequence anzulegen

### DECIMALDIGITS 
Deklarationsbereich  
[VAR_INPUT](#var_input)

Objektbereich  
[Parameter](#parameter)

Syntax  
```cpp
[DECIMALDIGITS(-1)]
```
Beschreibung  
DD (Decimal Digits) steht für die Anzahl der Dezimalstellen, die bei der Anzeige des Parameterwerts berücksichtigt werden sollen. Diese Anzahl wird üblicherweise im Parameter-Setup definiert und kann je nach Anwendung variieren. Wenn der Wert von DD auf 0 gesetzt wird, bedeutet dies, dass keine Nachkommastellen angezeigt werden sollen und der Wert als Ganzzahl dargestellt wird. Wenn DD auf -1 gesetzt wird, werden keine Nachkommastellen berücksichtigt, wenn DD auf 1 gesetzt wird, wird eine Dezimalstelle berücksichtigt und so weiter. Die Anzahl der Dezimalstellen ist wichtig, um eine korrekte und genaue Anzeige des Parameterwerts sicherzustellen. Wenn die Anzahl der Dezimalstellen nicht ausreichend ist, können wichtige Informationen verloren gehen oder ungenau dargestellt werden. Wenn die Anzahl der Dezimalstellen zu hoch ist, kann dies die Lesbarkeit des Wertes beeinträchtigen. Daher ist es wichtig, die Anzahl der Dezimalstellen sorgfältig zu definieren und zu überwachen, um eine genaue und lesbare Darstellung des Parameterwerts zu gewährleisten.

DD -1 (Input/Output)  
![HMI](images/HMIParameterDD_Default.png) ![HMI](images/HMIParameterDD_Default_Out.png)  
DD 0 (Input/Output)  
![HMI](images/HMIParameterDD_Zero.png) ![HMI](images/HMIParameterDD_Zero_Out.png)  
DD 1 (Input/Output)  
![HMI](images/HMIParameterDD_One.png) ![HMI](images/HMIParameterDD_One_Out.png)  

### DECLARATIONASINPUT
### DECLARATIONASINPUT
Deklarationsbereich  
[VAR_OUTPUT](#var_output)

Objektbereich  
[CMZ](#cmz)

Syntax  
```cpp
[DECLARATIONASINPUT(false)], [DECLARATIONASINPUT(true)]
```
Beschreibung  
Wenn Sie die CMZ als `True` deklarieren, wird sie als Hardware-Eingang deklariert und mit dem `AT %I*` Attribut  in der Programmierungslogik eingebunden. Dies bedeutet, dass die Variable ein Signal oder einen Wert von einem physikalischen Eingang des Systems empfängt, wie beispielsweise von einem Sensor oder einem Schalter.

![PLC](images/PLCCMZDeclerationAsHard.png)

### DISABLEAUTO
Deklarationsbereich  
[VAR_INPUT](#var_input)

Objektbereich  
[Parameter](#parameter)

Syntax
```cpp
[DISABLEAUTO(false)], [DISABLEAUTO(true)], 
```

Beschreibung
Die Funktion "Disable Input in Automatic" ermöglicht es, das Ändern von Eingabevariablen zu sperren, solange der Automatik-Modus aktiv ist. Dies bedeutet, dass Benutzer den Wert einer Eingabevariablen nicht manuell ändern können, solange das System im Automatik-Modus arbeitet. Diese Funktion ist besonders nützlich, um die Sicherheit und Integrität des Systems zu gewährleisten, da sie verhindert, dass Benutzer versehentlich den Betrieb des Systems beeinträchtigen, während es in einem automatisierten Betriebsmodus arbeitet. Wenn der Automatik-Modus deaktiviert ist, können Benutzer den Wert der Eingabevariablen wieder manuell ändern. Diese Funktion ist besonders hilfreich in industriellen Anwendungen, in denen es wichtig ist, dass das System sicher und zuverlässig funktioniert, auch wenn es von unterschiedlichen Benutzern betrieben wird.

### ERRORDELAY
Deklarationsbereich  
[VAR_OUTPUT](#var_output)

Objektbereich  
[CMZ](#cmz)

Syntax  
```cpp
[DECLARATIONASINPUT(false)], [DECLARATIONASINPUT(true)]
```
Beschreibung  
Wenn Sie die CMZ als `True` deklarieren, wird sie als Hardware-Eingang deklariert und mit dem `AT %I*` Attribut  in der Programmierungslogik eingebunden. Dies bedeutet, dass die Variable ein Signal oder einen Wert von einem physikalischen Eingang des Systems empfängt, wie beispielsweise von einem Sensor oder einem Schalter.

### GHOSTMODE

### GHOSTMODEDELAY

### HARDWAREINPUT
Deklarationsbereich  
[VAR_OUTPUT](#var_output)

Objektbereich  
[Zone In](#zone-in), [Zone InOut](#zone-inout)

Syntax  
```cpp
[DECLARATIONASINPUT(false)], [DECLARATIONASINPUT(true)]
```
Beschreibung  
Wenn Sie den Input als `true` deklarieren, wird sie als Hardware-Eingang deklariert und mit dem `AT %I*` Attribut in der Programmierungslogik eingebunden. Dies bedeutet, dass die Variable ein Signal oder einen Wert von einem physikalischen Eingang des Systems empfängt, wie beispielsweise von einem Sensor oder einem Schalter.

![Studio](images/StudioInputDeclarationAsHInputProperty.png)  
![PLC](images/PLCInputDeclarationAsHInputProperty.png)

### HARDWAREOUTPUT
Deklarationsbereich  
[VAR_INPUT](#var_input)

Objektbereich  
[Zone InOut](#zone-inout), [Zone Output](#zone-output)

Syntax
```cpp
[HARDWAREOUTPUT(false)], [HARDWAREOUTPUT(true)] 
```

Beschreibung
Wenn Sie den Output als `true` deklarieren, wird er als Hardware-Ausgang deklariert und mit dem `AT %Q*` Attribut in der Programmierungslogik eingebunden. Dies bedeutet, dass die Variable ein Signal oder einen Wert auf einem physikalischen Ausgang des Systems sendet, wie beispielsweise von einem Ventil oder einem Umrichter.

![Studio](images/StudioOutputDeclerationAsHardProperty.png)  
![PLC](images/PLCOutputDeclerationAsHOutput.png)

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

