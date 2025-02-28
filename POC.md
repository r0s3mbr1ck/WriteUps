
PoC_CVE_2022-26923
share_link
https://share.note.sx/vlbz1qjl#WjmrhXZyPDktrgFXyHbvt5W7TnykmhfhfQ/mUFEhlCw
share_updated
2024-10-21T19:57:27-03:00
Author
Alex Marano
Criado em
20/10/2024
Explicação
A escalação de privilégio se dá por meio da vulnerabilidade CVE 2022-26923 no *Active Directory Certificate Service* (AD CS) da Microsoft, que permite, que qualquer usuário do AD escale seus privilégios para o Administrador de Domínio em um único salto, ao explorar modelos de certificados mal configurados.

Por padrão, qualquer usuário que seja membro do grupo Usuários Autenticados (literalmente todas as contas do AD), pode registrar até 10 novas máquinas no domínio. Isso geralmente é usado em organizações para permitir que os usuários tragam seu próprio dispositivo (BYOD) e o registrem para uso no domínio. Isso em si não é realmente uma vulnerabilidade, mas levou a alguns vetores de escalação de privilégios.


https://msrc.microsoft.com/update-guide/vulnerability/CVE-2022-26923
Ambiente
Servidor Active Directory
Nome do Sistema Operacional: Microsoft Windows Server 2022 Datacenter
Versão do Sistema Operacional: 10.0.20348 N/A Compilação 20348
Tipo de Sistema: 64 bits
Hotfix: KB5004330, KB 5008223 e KB 5008995.
Virtualização: QEMU Virt-manager (Debian 12)
Domínio: CONTROLADOR.LOCAL
Endereço IP: 192.168.122.2
Credenciais: Administrator:ChangesMe123
Serviços Instalados: Active Directory Domain Server (AD CS) e Active Directory Certificate Services (AD CS)
Pasted image 20241020112928.png
Pasted image 20241020112955.png
Link da VM:
"https://archive.org/details/20348.169.210806-2348.fe-release-svc-refresh-server-eval-x-64-fre-en-us_202112"
Acesso em 27 de setembro de 2024.

Host8