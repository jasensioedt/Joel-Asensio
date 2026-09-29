```
dc=hospitaletfc, dc=es
 |--- ou=Departamentos
 |     |--- ou=Marketing
 |     |     |--- uid=usuario1
 |     |     |--- uid=usuario2
 |     |--- ou=RRHH
 |     |     |--- uid=usuario3
 |     |     |--- uid=usuario4
 |     |--- ou=Finanzas
 |     |     |--- uid=usuario5
 |     |     |--- uid=usuario6
 |     |--- ou=IT
 |     |     |--- uid=usuario7
 |     |     |--- uid=usuario8
 |     |--- ou=CuerpoTecnico
 |           |--- uid=usuario9
 |           |--- uid=usuario10
```

## DN de cada usuario

### Marketing
```
uid=usuario1,ou=Marketing,ou=Departamentos,dc=hospitaletfc,dc=es
uid=usuario2,ou=Marketing,ou=Departamentos,dc=hospitaletfc,dc=es
```

### RRHH
```
uid=usuario3,ou=RRHH,ou=Departamentos,dc=hospitaletfc,dc=es
uid=usuario4,ou=RRHH,ou=Departamentos,dc=hospitaletfc,dc=es
```

### Finanzas
```
uid=usuario5,ou=Finanzas,ou=Departamentos,dc=hospitaletfc,dc=es
uid=usuario6,ou=Finanzas,ou=Departamentos,dc=hospitaletfc,dc=es
```

### IT
```
uid=usuario7,ou=IT,ou=Departamentos,dc=hospitaletfc,dc=es
uid=usuario8,ou=IT,ou=Departamentos,dc=hospitaletfc,dc=es
```

### CuerpoTecnico
```
uid=usuario9,ou=CuerpoTecnico,ou=Departamentos,dc=hospitaletfc,dc=es
uid=usuario10,ou=CuerpoTecnico,ou=Departamentos,dc=hospitaletfc,dc=es
```
