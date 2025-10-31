#!/bin/bash

if [ $(id -u) -ne 0 ]; then
    echo -e "Debes ser root"
    exit
fi

read -p "Introduce la dirección IP: " ip

while true; do
    echo -e "\n1) Escaneo Normal"
    echo "2) Escaneo de servicios y versiones"
    echo "3) Buscar CVE (Vulnerabilidades)"
    echo "4) OSINT (Serachsploit + Shodan)"
    echo "5) Salir"
    read -p "Seleccione una opción: " opcion
    case $opcion in

    1)
        clear && echo "Escaneando..." && nmap -p- -Pn --open $ip >Escaneo_normal.txt && echo -e "Reporte guardado en el fichero Escaneo_normal.txt"
        grep "open" Escaneo_normal.txt
        exit
        ;;

    2)
        clear && echo "Escaneando..." && nmap -sV -sC $ip > Escaneo_servicios.txt && echo -e "Reporte guardado en el fichero Escaneo_servicios.txt"
        exit
        ;;

    3)
        clear && echo "Buscando CVE ..." && nmap -sV --script=vulners --script-args=vulners.showall=1 $ip > Escaneo_cves.txt && echo -e "Reporte guardado en Escaneo_cves.txt"
        exit
        ;;

    4)
        clear && echo "Ejecutando nmap para detectar servicios..." && nmap -sV $ip > tmp_nmap.txt > /dev/null
        echo "Se han detectado servicios:" && grep "open" tmp_nmap.txt

        # Buscar exploits con Searchsploit
        echo -e "\nExploit sugeridos (Searchsploit):"
        grep "open" tmp_nmap.txt | awk '{print $3, $4}' | while read svc ver; do
            echo -e "\nServicio: $svc $ver"
            searchsploit $svc | head -n 20
        done

        # Consultar Shodan (solo info básica)
        if [ -z "$SHODAN_API_KEY" ]; then
            echo -e "\nSHODAN_API_KEY no definido, no se puede consultar Shodan."
        else
            echo -e "\nInformación básica de Shodan:"
            curl -s "https://api.shodan.io/shodan/host/$ip?key=$SHODAN_API_KEY" | grep -E '"ip_str"|"ports"'
        fi

        rm -f tmp_nmap.txt
        exit
        ;;

    5)
        break
        ;;
    *)
        echo -e "No se ha encontrado el parametro"
        ;;
    esac
done

