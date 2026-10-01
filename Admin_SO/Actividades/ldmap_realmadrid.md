```
dc=hospitaletfc, dc=es
 |--- ou=Direccion
 |     |--- uid=jmartinez
 |     |--- uid=mgarcia
 |--- ou=Departamentos
 |     |--- ou=Marketing
 |     |     |--- uid=lperez
 |     |     |--- uid=asanchez
 |     |--- ou=RRHH
 |     |     |--- uid=cmoreno
 |     |     |--- uid=nfernandez
 |     |--- ou=Finanzas
 |     |     |--- uid=rjimenez
 |     |     |--- uid=pvidal
 |     |--- ou=IT
 |           |--- uid=jroca
 |           |--- uid=anavarro
 |     
 |--- ou=Deportivo
 |     |--- ou=CuerpoTecnico
 |     |     |--- uid=aruiz
 |     |     |--- uid=dortega
 |     |--- ou=Jugadores
 |     |     |--- uid=jmolina
 |     |     |--- uid=mvazquez
 |     |--- ou=Coordinacion
 |     |     |--- uid=sramos
 |     |     |--- uid=cgil
 |     |--- ou=FutbolBase
 |     |     |--- uid=eblanco
 |     |     |--- uid=fprat
 |     |--- ou=GestionDeportiva
 |           |--- uid=gsoler
 |           |--- uid=hcampos
 |--- ou=Instalaciones
       |--- ou=Mantenimiento
       |     |--- uid=ibenitez
       |     |--- uid=jflores
       |--- ou=Seguridad
             |--- uid=kpuig
             |--- uid=lmora
```

## DN de cada usuario

### Direccion
```
uid=jmartinez,ou=Direccion,dc=hospitaletfc,dc=es
uid=mgarcia,ou=Direccion,dc=hospitaletfc,dc=es
```

### Marketing
```
uid=lperez,ou=Marketing,ou=Departamentos,dc=hospitaletfc,dc=es
uid=asanchez,ou=Marketing,ou=Departamentos,dc=hospitaletfc,dc=es
```

### RRHH
```
uid=cmoreno,ou=RRHH,ou=Departamentos,dc=hospitaletfc,dc=es
uid=nfernandez,ou=RRHH,ou=Departamentos,dc=hospitaletfc,dc=es
```

### Finanzas
```
uid=rjimenez,ou=Finanzas,ou=Departamentos,dc=hospitaletfc,dc=es
uid=pvidal,ou=Finanzas,ou=Departamentos,dc=hospitaletfc,dc=es
```

### IT
```
uid=jroca,ou=IT,ou=Departamentos,dc=hospitaletfc,dc=es
uid=anavarro,ou=IT,ou=Departamentos,dc=hospitaletfc,dc=es
```

### CuerpoTecnico
```
uid=aruiz,ou=CuerpoTecnico,ou=Deportivo,dc=hospitaletfc,dc=es
uid=dortega,ou=CuerpoTecnico,ou=Deportivo,dc=hospitaletfc,dc=es
```

### Jugadores
```
uid=jmolina,ou=Jugadores,ou=Deportivo,dc=hospitaletfc,dc=es
uid=mvazquez,ou=Jugadores,ou=Deportivo,dc=hospitaletfc,dc=es
```

### Coordinacion
```
uid=sramos,ou=Coordinacion,ou=Deportivo,dc=hospitaletfc,dc=es
uid=cgil,ou=Coordinacion,ou=Deportivo,dc=hospitaletfc,dc=es
```

### FutbolBase
```
uid=eblanco,ou=FutbolBase,ou=Deportivo,dc=hospitaletfc,dc=es
uid=fprat,ou=FutbolBase,ou=Deportivo,dc=hospitaletfc,dc=es
```

### GestionDeportiva
```
uid=gsoler,ou=GestionDeportiva,ou=Deportivo,dc=hospitaletfc,dc=es
uid=hcampos,ou=GestionDeportiva,ou=Deportivo,dc=hospitaletfc,dc=es
```

### Mantenimiento
```
uid=ibenitez,ou=Mantenimiento,ou=Instalaciones,dc=hospitaletfc,dc=es
uid=jflores,ou=Mantenimiento,ou=Instalaciones,dc=hospitaletfc,dc=es
```

### Seguridad
```
uid=kpuig,ou=Seguridad,ou=Instalaciones,dc=hospitaletfc,dc=es
uid=lmora,ou=Seguridad,ou=Instalaciones,dc=hospitaletfc,dc=es
```
