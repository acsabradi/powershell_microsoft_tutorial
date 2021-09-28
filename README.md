# PowerShell Microsoft Tutorial
[link](https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.1)

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

**TODO**: Active Directory fejezet