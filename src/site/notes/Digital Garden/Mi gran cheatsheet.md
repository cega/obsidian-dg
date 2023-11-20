---
{"dg-publish":true,"permalink":"/digital-garden/mi-gran-cheatsheet/"}
---


# bash

En `piler`, luego de importar mails para ser archivados, quedan en la carpeta `import` un cierto remanente que no pudo ser procesado.
Uno de las razones identificadas es la presencia dentro de los encabezados del mensaje de caracteres inválidos.
Por ahora encontré gran número de mensajes que viene con el carácter `\xa0` en el campo `to`.

Para separarlos, utilizo este `script`:

Nota:  [Ver post en stackoverflow](https://stackoverflow.com/a/34195247/2494813)


```bash
BAD_PATH=./bad_$(date  +'%y%m%d_%H%M')
mkdir -p ${BAD_PATH}
for f_pattern in '*zimbra' '*mailspy' '*msg'
do
    if compgen -G "./${f_pattern}" > /dev/null; then
        echo ${f_pattern}
        grep -q $'\xa0' ${f_pattern} && mv ${f_pattern} ${BAD_PATH} || echo "No encontrado"
    fi
done

```

* Cargar en el entorno las variables de un archivo

```bash
set -o allexport; source .env; set +o allexport
```
# python

# typescript

# postgres

### [How to turn JSON array into Postgres array?](https://dba.stackexchange.com/questions/54283/how-to-turn-json-array-into-postgres-array)

Para Postgres 9.4+:
>* json_array_elements_text(json)
>* jsonb_array_elements_text(jsonb)

# vim

* Buscar carácter hexadecimal: `:\%xa0`
* Ver el archivo en hexadecimal: `%!xxd`

# ssh
[Troubleshooting Linux SSH](https://tanelpoder.com/posts/troubleshooting-linux-ssh-logon-delay-always-takes-10-seconds/)
> Resumen: Es posible que algunos problemas se presenten al intentar el SSH login con public keys por una mala resolución de DNS inverso en el servidor.

# otros

# openssl

* Ver el certificado en formato texto
	```bash
	openssl x509 -noout -text -in <CERTIFICADO>
	```
* Ver los certificado que presenta un `url`
	Nota: el url no debe incluir el `schema` ( http, https, etc.)
	```bash
	openssl s_client -connect <url:port> -showcerts
	```
	por ejemplo
	```bash
	openssl s_client -connect zimbra.constable.com.ar:443 -showcerts
	```
