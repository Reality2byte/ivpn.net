---
title: IVPN para macOS - Aplicación VPN de código abierto para tu Mac
description: IVPN para macOS te ofrece una protección integral contra filtraciones de privacidad con el firewall IVPN, conexión automática en redes Wi-Fi inseguras y Multi-hop.
h1: IVPN para macOS
subtitle: Compatible con macOS 10.14+
url: /es/apps-macos/
platform: macos
layout: apps
image: apps/macos-app
releases: [{
    cta: Download,
    downloads: [
        {
            cta: Intel,
            url: "https://repo.ivpn.net/macos/bin/IVPN-3.14.34.dmg"
        },
        {
            cta: Apple Silicon,
            url: https://repo.ivpn.net/macos/bin/IVPN-3.14.34-arm64.dmg
        }
    ],
    github: https://github.com/ivpn/desktop-app,
    changelog: https://github.com/ivpn/desktop-app/blob/master/CHANGELOG.md,
    checksum: [
        {
            title: SHA256 Intel,
            value: a1024b219174d3fccaac592be8129cc43f9690fb1a6dcb3ecbcad6b1e610a4aa
        },
        {
            title: SHA256 Apple Silicon,
            value: 08cfa04033591a7320ed2c06697ecc34e9666e9a0f026babc7c4b665bf4c0303
        }
    ]
}]
---
## Funciones

- Protocolos WireGuard y OpenVPN.
- GUI o CLI (interfaz de línea de comandos).
- Controles de privacidad de WireGuard - Define un programa de rotación automática de claves y direcciones IP.
- AntiTracker que bloquea anuncios, adware, páginas web maliciosas y rastreadores de recopilación de datos.
- Firewall/killswitch - Posibilidad de configurarlo a demanda o siempre activo. Ofrece protección integral contra filtraciones DNS, IPv6, WebRTC y desconexiones.
- Capacidad para definir redes Wi-Fi de confianza y crear reglas para la conexión/desconexión automática de la VPN.
- Rutas VPN de Multi-hop. Conéctate a través de múltiples servidores en jurisdicciones diferentes para mejorar la privacidad.
- Permitir el tráfico LAN cuando estés conectado a la VPN.
- Función de pausado para cuando sea necesario deshabilitar temporalmente la conexión VPN.
- Opción Obfsproxy para eludir la censura.
- Servidores DNS personalizados, con DoH.
- Actualización automática.
- Conexión automática de inicio y al unirse a una red Wi-Fi insegura.

## Configuración manual

Si prefieres no utilizar la aplicación IVPN, sigue la guía de configuración correspondiente a continuación.

- [WireGuard](/setup/macos-wireguard/)
- [Tunnelblick (OpenVPN)](/setup/macos-openvpn-tunnelblick/)  
- [IPSec con IKEv2](/setup/macos-ipsec-with-ikev2/)   

## Descarga una versión anterior

Download [IVPN-2.12.17.dmg](https://cdn.ivpn.net/releases/osx/IVPN-2.12.17.dmg)  
SHA256: 0fd09967482f53c801dc55eaf23a88ad341da37f58d70d9c9e24c2e5aeb36c22  
