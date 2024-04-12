# P4 Programming Tutorial on Netronome SmartNICs
This tutorial provides a basic introduction to P4 (Programming Protocol-independent Packet Processors) programming on Netronome SmartNICs. Here you will find all the steps for installing and setting up the development environment, as well as presenting, implementing, and running a simple P4 program on a Netronome SmartNIC.

The tutorial is designed in a simple and straightforward manner, aiming to abstract away the common complexities found in hardware programming. The goal is to provide a quick learning curve by covering only the essential topics up to the implementation of the first program.

# Prerequisites
- Advanced knowledge of computer networks.
- Familiarity with Linux and command line.
- Basic understanding of the P4 language.
-Access to a Netronome SmartNIC (in this tutorial, we will use the Netronome Agilio CX 2x10Gb).

# Installation and Environment Setup
To install the necessary drivers and modules and configure the environment, follow the instructions below:

- Access the Agilio-P4-SmartNIC directory.
- Follow the presented tutorial to install the required drivers and modules.

By following these steps, you will be ready to start developing and running your P4 programs on Netronome SmartNIC. In the instalation directory you'll find a lot of netronome related documents, that can be usefull in case of you want do deep into this technology.

## Details about Netronome architecture

The Netronome SmartNIC uses single-root input/output virtualization (SRIOV), which enables virtual functions (VFs) to be created from a physical function (PF). The VFs thus share the resources of a PF, while VFs remain isolated from each other. The isolated VFs are typically assigned to virtual machines (VMs) on the host. In this way, the VFs allow the VMs to directly access the PCI device, thereby bypassing the host kernel. In this tutorial, we have two physical (p0, p1) and four virtual interfaces (Vf0\_1 to Vf0\_5). We will work with a P4 program that implements simple IPv6 forwarding, which can be found at IPv6Forwarding.

# Deployment

Here we'll show how to deploy, configure, and debug your programs on Netronome SmartNICs. We are taking into consideration that you already clone this repository and that your environment is already installed and configured.

Once you already have your netronome installed and configured on you machine, you need to locate the src/p4-16 folder, inside the Agilio-P4-SmartNIC directory. This folder contains all the programs that you can run on your Netronome SmartNIC. Você poderá inclusive encontrar programas diferentes deste que vamos testar. Para começar, certifique-se de que esteja dentro da pasta p4-16, e então crie uma pasta chamada SimpleIPv6. Após isto, copie o conteúdo da pasta IPv6Forwarding para a pasta que acabou de criar:
```
mkdir SimpleIPv6
cd SimpleIPv6
cp /IPv6Forwarding/ipv6_forwarding.p4 .
cp /IPv6Forwarding/user_config.json .
```

Para programação em P4 usando netronome, estes dois arquivos são tudo o que é necessário. O arquivo ipv6_forwarding.p4 representa o nosso programa propriamente disto. E o user_config.json é o arquivo que irá popular as tabelas do plano de controle. Como este tutorial é para aqueles que já conhecem a linhaguem de programação P4, não vamos entrar em detalhes. No entanto, um problema muito comum daqueles que estão começando a programar usando smartnics Netronome é em como realizar a atribuição de interfaces na tabela do plano de dados. Por exemplo, olhando o código P4, podemos encontrar seguinte ação:

```action ipv6_forward (macAddr_t dstAddr, egressSpec_t port) {
        standard_metadata.egress_spec = port;
        hdr.ethernet.dstAddr = dstAddr;
```
Esta ação receberá uma porta com parametro, que será a porta de saída do pacote em questão. Quando estamos trabalhando com smartnics Netronome, temos as interfaces físicas e as virtuais. You can configure this setting by editing the following file: `/lib/systemd/system/nfp-sdk6-rte.service`. locate and change the following line.

`Environment=NUM_VFS=4`

Com esta configuração, você terá 4 interfaces virtuais, chamadas de VFs. E elas podem ser instanciadas nas tabelas do plano de controle como "v0.0", "v0.1", "v0.2" e v0.3". Caso queira utilizar as interfaces físicas, deverá chama-las de "p0" e "p1". 

No arquivo user_config.json podemos encontrar exemplos de como popular as tabelas do plano de controle utilizando a nomenclatura supracitada. 

Bom, agora que já copiamos os dois arquivos necessários, devemos chamar o compilardor passando o arquivo P4 como parametro para que seja criado um firmware. Depois vamos implantar o firmware na placa e por ultimo vamos popular as tabelas do plano de controle. Para isto, dê os seguintes comandos:

```
sudo /opt/netronome/p4/bin/./nfp4build --nfp4c_p4_version 16 --no-debug-info -p out -o firmware.nffw -l lithium -4 ipv6_forward.p4
sudo /opt/netronome/p4/bin/./rtecli design-load -f firmware.nffw -p out/pif_design.json
sudo /opt/netronome/p4/bin/rtecli config-reload -c user_config.json
```

Para debugar, você poderá olhar os logs em `/var/log/nfp-sdk6-rte.log`. Aqui serão encontrados todos os erros referentes ao serviço, implantação e utilização. Ou seja, caso tenha problemas em inicializar o serviço ou implantação do programação, poderá encontrar os logs e entender melhor o que está acontecendo. 

Obs: após a criação do firmware, você notará que muitos arquivos foram criados. Não se preocupe, é normal.

Sobre a configuração do plano de controle, if you want, you can check the configured rules.
```
sudo /opt/netronome/p4/bin/./rtecli tables -i 0 list-rules
```

Agora a sua placa está devidamente configurada com um código simples, que encaminhará dados de uma porta virtual para outra usando IPv6. Você pode utilizar os códigos python send_pkt.py e receive.py, onde o primeiro enviará um pacote da interface v0.0 para a interface v0.3. E o segundo programa realizará a captura do pacote na interface de destino.

Para executar os programas de teste, abra dois shells, navegue até este diretório e no primeiro shell de o comando `python3 receive.py`. O programa será executado e ficará na espera de qualquer pacote recebido na interface v0.3. Após isto, no segundo terminal, execute `python3 send_pkt.py`. Após isto, um pacote será enviado na interface v0.0 com destino ao v0.3. A execução destes programas python dependem da configuração de cada computador e da versão do python.

Caso a execução dos programas tenham sido bem sucedidos, parabens, você concluiu este tutorial e o programa e está pronto para o próximo passo. Caso tenha interesse em se aprofundar mais e realizar funções mais avanças, talvez [este repositório]{https://github.com/guimvmatos/P4-INCA} lhe interessará. Nele há também um tutorial para implantação de um programa mais avançado, utilizando SRv6 para conectar o trafego de várias maquinas virtuais. Fique a vontade para conhecer e tentar.







## Steps

1. INCA
First, you must configure your card to have at least 7 logical interfaces (VFs). You can configure this setting by editing the following file: `/lib/systemd/system/nfp-sdk6-rte.service`. locate and change the following line.

`Environment=NUM_VFS=7`

Once this is done, proceed with the inca code P4. You have to clone this repository inside your source codes of netronome, generally its inside `path_for_agilio/src/p4-16`.

```
git clone git@github.com:guimvmatos/P4-INCA.git
cd P4-INCA
sudo /opt/netronome/p4/bin/./nfp4build --nfp4c_p4_version 16 --no-debug-info -p out -o firmware.nffw -l lithium -4 ipv6_forward.p4
sudo /opt/netronome/p4/bin/./rtecli design-load -f firmware.nffw -p out/pif_design.json
sudo /opt/netronome/p4/bin/rtecli config-reload -c user_config.json
```

If you want, you can check the configured rules.
```
sudo /opt/netronome/p4/bin/./rtecli tables -i 0 list-rules
```

Now your netronome is configured with INCA. Let's proceed with configurations of RAN, UPF and others VMs.

2. RAN / UPF

Follow the instructions of [P4-BMv2-RAN-UPF Repository](https://github.com/guimvmatos/P4-BMv2-RAN-UPF)

After this, you'll have two VMs running and linked with INCA.

3. VFs topology

Follow the instructions of [SFC-SRv6-topology](https://github.com/guimvmatos/SFC-SRv6-topology)


Now, you have all topology running. You can ping from clientVlc to dashServer with `ping6 fc20::2`. You can capture the packets passing through NFVs with tcpdump.

