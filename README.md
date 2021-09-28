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