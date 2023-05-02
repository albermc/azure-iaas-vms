## Paso 1: Crear un grupo de recursos.

`create az group --name platziBalancer --location eastus`

## Paso 2: Crear una IP Pública.

`create az network public-ip create -g platziBalancer --name myPublicIP`

## Paso 3: Crear un balanceador de cargas.

`az network lb create -g platziBalancer -n myLoadBalancer --frontend-ip-name myFrontEndPool --backend-pool-name myBackEndPool --public-ip-address myPublicIP`

## Paso 4: Configurar un seguimiento de desempeño del balanceador de cargas. Agregamos una regla de seguridad para monitorear todo el comportamiento de nuestro balanceador.

`az network lb probe create -g platziBalancer --lb-name myLoadBalancer --name myHealthProbe --protocol tcp --port 80`

## Paso 5: Establecer una regla de red en el balanceador de cargas.

`az network lb rule create -g platziBalancer --lb-name myLoadBalancer -n myLoadBalancerRule --protocol tcp --frontend-port 80 --backend-port 80 --frontend-ip-name myFrontEndPool --backend-pool myBackEndPool --probe-name myHealthProbe`

## Paso 6: Crear una red virtual.

`az network vnet create -g PlatziBalancer -n myVnet --subnet-name mySubnet` 

## Paso 7: Crear un grupo de seguridad de red (NSG).

`az network nsg create -g PlatziBalancer -n myNetworkSecurityGroup`

## Paso 8: Establecer una regla de entrada en el grupo de seguridad de red.

`az network nsg rule create -g PlatziBalancer --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRule --priority 1001 --protocol tcp --destination-port-range 80`

## Paso 9: Crear 3 interfaces de red, una para cada máquina virtual.

for i in `seq 1 3`; do
> az network nic create  -g PlatziBalancer --name myNic$i --vnet-name myVnet --subnet mySubnet --network-security-group myNetworkSecurityGroup --lb-name myLoadBalancer --lb-address-pools myBackEndPool
> done

## Paso 10: Crear un conjunto de disponibilidad.

`az vm availability-set create -g PlatziBalancer --name myAvailabilitySet`

## Paso 11: Crear un archivo de instalación y configuración de arranque para las máquinas virtuales. Este archivo lo puedes encontrar en la sección de recursos de la clase.


## Paso 12: Crear 3 máquinas virtuales.
for i `seq 1 3`; do
> az vm create -g PlatziBalancer --name myVM$i --availability-set myAvailabilitySet --nics myNic$i --image UbuntuLTS --admin-username azureuser --generate-ssh-key --custom-data step-11.yml --no-wait
> done

## Paso 13: Obtener la IP del balanceador de cargas. Puedes acceder a ella desde el portal de Azure o la línea de comandos.
`az network public-ip show -g PlatziBalancer --name myPublicIP --query [ipAddress] --output tsv`

## Paso 14: Usar la IP del balanceador de cargas para probar que la arquitectura de nuestra aplicación funciona correctamente.