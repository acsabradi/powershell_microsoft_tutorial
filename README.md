# PowerShell Microsoft Tutorial
**[link](https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.1)**

## Help System

A PowerShell parancsoknak (*cmdlet*) `Ige-Főnév` alakjuk van.
```ps
Get-Help
```

A parancsokról ad segítséget. 
Először a parancsok nevében keres, ha nem talál semmit, akkor a help szövegben is keres.

```ps
Get-Help -Name Get-Help
```

Segítség kérése a `Get-Help` parancshoz. Ha először van ez a parancs végrehajtva, akkor először le fogja tölteni a help szövegeket.

```ps
Get-Help -Name Get-Help -Full
```

Teljes help megjelenítése.

```ps
Get-Help -Name Get-Help -Parameter Name
```

Egy parancs paraméterére keresés.

```ps
Get-Help -Name Get-Help -Examples
```

Példakódra keresés.

```ps
Get-Help -Name Get-Help -ShowWindow
```

Külön ablakban jeleníti meg a help-et.

```ps
Get-Help -Name *process*
```

Nem teljes névre is rá lehet keresni.

```ps
Get-Help -Name *-process*
```

Ha kötőjellek kezdődik a szöveg, amire rá akarunk keresni, akkor mindenképp kell elé egy wildcard, különben paraméternek fogja a PowerShell tekinteni.

```ps
help About_*
```

Részletes tutorial-ok listája. A téma nevének megadásával (pl. `help About_ForEach`) a tutorial-t nyitjuk meg.

```ps
Get-Command -Noun Process
```

Minden parancs kilistázása, aminek a főnév (*noun*) része *Process*, tehát process-ekkel lép kapcsolatba.

```ps
Get-Command -Name *service* -CommandType Cmdlet, Function, Alias
```

Parancs típusra szűrés. Így nem fognak megjelenni a listában a nem-natív PowerShell parancsok.

```ps
Update-Help
```

Help rendszer frissítése.

### Mi a különbség a `Get-Help` és a `Get-Command` között?

A `Get-Help` egy felhasználóbarát szöveg-alapú leírást ad, a `Get-Command` objektumot ad vissza, amit tovább használhatunk a logikában.

## Objektumok

### Property-k

```ps
Get-Service -Name w32time | Get-Member
```

A `Get-Service` egy objektumot ad át a `Get-Member`-nek, ami kilistázza az objektum adatait (pl. property-k, metódusok).

Az eredmény első sorában mindig a kapott objektum típusa szerepel, ami a fenti sort lefuttatva `System.ServiceProcess.ServiceController`.

```ps
Get-Command -ParameterType ServiceController
```

Kilistázza azokat a parancsokat, melyek `ServiceController` típusú objektumot elfogad inputként.

```ps
Get-Service -Name w32time | Select-Object -Property *
```

A `Get-Member` alapértelmezetten nem listáz ki mindent, ehhez a `Select-Object`-et kell használni. A fenti sor kilistáz minden property-t.

```ps
Get-Service -Name w32time | Select-Object -Property Status, Name, DisplayName, ServiceType
```

Célzottan is ki lehet listázni property-ket.

```ps
Get-Service -Name w32time | Select-Object -Property Status, DisplayName, Can*
```

Wildcard itt is használható.

### Metódusok

```ps
Get-Service -Name w32time | Get-Member -MemberType Method
```

Az objektum metódusainak listázása.

```ps
(Get-Service -Name w32time).Stop()
```

Egy objektum-metódus hívása.

A metódusok hívását érdemes kerülni és inkább a natív `Ige-Főnév` alakú parancsokat használni. Előfordulhat viszont, hogy egy adott típushoz nincsen *cmdlet*, ilyenkor használjunk egy megfelelő metódust.

```ps
Get-Service -Name w32time | Start-Service -PassThru
```

Alapesetben a `Start-Service` parancs nem ad vissza semmit, viszont ez felülírható a `PassThru` paraméterrel. Ilyenkor visszaadja azt az objektumot, amit módosított. A konzolon azt kellene látnunk, hogy a service *Running* státuszban van.

```ps
Start-Service -Name w32time -PassThru | Get-Member
```

A `Get-Member` egy objektumot vár, viszont a `Start-Service` alapból nem ad vissza semmit. A `PassThru` paraméterrel itt is megoldható, hogy visszaadja az objektumot, amit elindított.

### Active Directory

```ps
Get-Command -Module ActiveDirectory
```

Az `ActiveDirectory`-ba tartózó összes parancs kilistázása.

```ps
Get-ADUser -Identity mike | Get-Member
```

A `Get-ADUser` parancs az `ActiveDirectory` modul része. A parancs ige főnév része `AD`-val kezdődik, elkerülve az esetleges névütközéseket.

Az objektumban lévő elemek csak egy részét kaptuk meg ezzel a cmdlet-el.

```ps
Get-ADUser -Identity mike -Properties * | Get-Member
```

A `Properties` paraméternek át kell adni egy wildcard-ot, ha minden elemet látni akarunk.

Teljesítmény megfontolások miatt ad vissza a default beállítás korlátozott számú elemet.

```ps
$Users = Get-ADUser -Identity mike -Properties *
```

A lekérdezések eredményét változóban tárolhatjuk. A változó tartalma nem fog változni, ha valami módosult az Active Directory-ban.

```ps
$Users | Get-Member
```

A változót átadhatjuk egy cmdlet-nek.

```ps
$Users | Select-Object -Property Name, LastLogonDate, LastBadPasswordAttempt
```

A `Select-Object` cmdlet-el property-ra is szűrhetünk.

```ps
Get-ADUser -Identity mike -Properties LastLogonDate, LastBadPasswordAttempt
```

A `Get-ADUser` cmdlet-nek is megadhatjuk, hogy melyik property-kre vagyunk kíváncsiak.

## Pipeline használata

### One-liner parancsok

```ps
Get-Service |
  Where-Object CanPauseAndContinue -eq $true |
    Select-Object -Property *
```

Lekérjük az összes service-t, majd kiszűrjük azokat, melyeknek a `CanPauseAndContinue` property-je `true`, majd kilistázuk ezen servive-ek összes property-jét.

A `|` *pipe* szimbólummal adhatjuk át egy cmdlet kimenetét egy másik cmdlet bemenetére, így írható egy *one-liner* parancs. A sor a következő karaktereknél törhető: `| , [ { ( ; = ' "`

```ps
Get-Service -Name w32time |
  Select-Object -Property *
```

`w32time` service lekérése, majd annak össze property-jének kilistázása.

```ps
$Service = 'w32time'; Get-Service -Name $Service
```

Ez nem *one-liner* parancs, hanem kettő parancs egy `;` szimbólummal elválasztva.

### Bal oldalról szűrés (filtering left)

```ps
# Forrásnál szűrés
Get-Service -Name w32time

# Minden service lekérése, majd azok szűrése
Get-Service | Where-Object Name -eq w32time
```

A két parancs ugyanazt az eredményt adja, a `w32time` service-t. Viszont az első parancs a forrásnál (balról) szűr, a második lekérdez minden service-t, majd azokat szűri név alapján, ezért az első parancs a gyorsabb.

```ps
Get-Service |
  Select-Object -Property DisplayName, Running, Status |
    Where-Object CanPauseAndContinue
```

Szűrésnél figyelni kell a parancsok sorrendjére. A példában lekérdezünk minden service-t, ebből vesszük azon property-ket, melyeket a `-Property` paraméternek adunk át, majd rászűrünk a `CanPauseAndContinue` property-re. Nem fogunk semmilyen eredményt se kapni, mivel mire a bemenet eléri a `Where-Object` cmdlet-et, addigra már kiszűrtük azt a property-t, amire szűrnénk ott.

```ps
Get-Service |
  Where-Object CanPauseAndContinue |
    Select-Object -Property DisplayName, Running, Status
```

Erre az a megoldás, hogy a két cmdlet sorrendjét felcseréljük, így már kapunk eredményt.

### Pipeline

```ps
help Stop-Service -Full

# Az output egy részlete

INPUTS
    System.ServiceProcess.ServiceController, System.String
        You can pipe a service object or a string that contains the name of a service
        to this cmdlet.
```

A *help* szerint a `Stop-Service` cmdlet egy `String` vagy `ServiceController` objektumot képes fogadni egy pipeline-ban. Ahhoz, hogy megtudjuk, pontosan melyik paraméter fogadja ezeket a bemeneteket, át kell nézni a paraméterek leírásait a *help*-ben:

```
-DisplayName <String[]>
    Specifies the display names of the services to stop. Wildcard characters are
    permitted.

    Required?                    true
    Position?                    named
    Default value                None
    Accept pipeline input?       False
    Accept wildcard characters?  false

-InputObject <ServiceController[]>
    Specifies ServiceController objects that represent the services to stop. Enter a
    variable that contains the objects, or type a command or expression that gets the
    objects.

    Required?                    true
    Position?                    0
    Default value                None
    Accept pipeline input?       True (ByValue)
    Accept wildcard characters?  false

-Name <String[]>
    Specifies the service names of the services to stop. Wildcard characters are
    permitted.

    Required?                    true
    Position?                    0
    Default value                None
    Accept pipeline input?       True (ByPropertyName, ByValue)
    Accept wildcard characters?  false
```

A `DisplayName` paraméter nem fogad el input-ot pipeline-ból, az `InputObject` elfogad `ServiceController` input-ot a pipeline-ból *ByValue* (érték szerint), a `Name` bemenet `String`-et fogad el *ByValue* és *ByPropertyName* (property név szerint). Ha a paraméter kétféleképp fogad el bemenetet, akkor mindig a *ByValue*-val fog először próbálkozni.

A *ByValue* azt jelenti, hogy ha egy `ServiceController` objektumot adok át a `Stop-Service` cmdlet-nek a pipeline-ban, akkor az az `InputObject` paraméter bemenete lesz. Ugyanez igaz egy `String` bemenetre és a `Name` paraméterre.

A *ByPropertyName* annyit tesz, hogy ha egy olyan tetszőleges objektumot adok át, ami tartalmaz egy `Name` property-t, akkor annak értéke lesz a `Name` paraméter bemenete.

```ps
Get-Service -Name w32time | Stop-Service
```

Mivel a `Get-Service` kimenete egy `ServiceController`, ezért az a pipeline-ban továbbadható a `Stop-Service`-nek, és így az `InputObject` bemenete lesz *ByValue*.

```ps
'w32time' | Stop-Service
```

A `Name` paraméternek adunk bemenetet *ByValue*.

```ps
$CustomObject = [pscustomobject]@{
	Name = 'w32time'
}

$CustomObject | Stop-Service
```

Egy általunk létrehozott objektumot adunk a `Stop-Service` bemenetére, így a `Name` paramétert hajtjuk meg *ByPropertyName*.

Ha az objektumban nem lenne `Name` property, akkor hibát kapnánk a pipeline futtatásakor. Ilyen esetben átalakíthatjuk a bemenetet úgy, hogy azt már elfogadja a `Stop-Service`:

```ps
$CustomObject = [pscustomobject]@{
  	Service = 'w32time'
}

$CustomObject |
  	Select-Object -Property @{name='Name';expression={$_.Service}} |
    	Stop-Service
```

Egyéb módon is lehet input-ot adni egy paraméterre:

```ps
'Background Intelligent Transfer Service', 'Windows Time' |
	Out-File -FilePath $env:TEMP\services.txt
	
Stop-Service -DisplayName (Get-Content -Path $env:TEMP\services.txt)
```

Először egy fájlba kiírjuk azokat a service-eket, melyeket le akarunk állítani, majd ugyanezt a fájlt beolvassuk, mikor át kell adni input-ot a `DisplayName` paraméternek.