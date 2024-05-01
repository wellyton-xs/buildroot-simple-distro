# como criar um sistema linux com buildroot

Esse artigo irá explicar o básico de como você pode
utilizar o buildroot para construir o seu próprio sistema
linux de forma fácil.

## o que é o buildroot

O buildroot é um projeto que visa facilitar a construção
de sistemas embarcados utilizando o linux, já deu pra entender
que nesse artigo iremos fazer algo um pouco diferente do que 
é o objetivo original do buildroot, mas isso será feito apenas
para aprendizado.

O site do buildroot possui uma ampla e vasta documentação. Se 
por acaso for do seu interesse, eis aqui o link:

https://buildroot.org/

## o que é um sistema embarcado

não há uma definição muito clara do que exatamente é um sistema
embarcado, mas dá pra se dizer que é um software que está bastante
ligado com o hardware com o qual ele está inserido. Ou seja, o sistema
depende daquele hardware e o hardware depende daquele sistema. Outra 
característica que dá pra citar é que geralmente os sistemas embarcados
eles são feitos para cobrir objetivos específicos e não são de propósitos
gerais. 

Um exemplo disso é aqueles sistemas básicos que vinham nas televisões antes
das smartTV existirem, trazendo o android para dentro delas.

Dava para você ver vídeos e outras coisas através de um pendrive e tinha 
um menu com algumas opções que a TV disponibilizava, aquele sistema só
existe para cumprir as funções daquela TV.

## mão na massa

Já deve ter visto que tenho os arquivos "pré-cozidos" no repositório,
mas não seja um molenga e tente fazer do zero seguindo o passo a passo,
isso vai tornar o seu aprendizado mais rico e você verá que será um processo
bem divertido!

faça o download do buildroot no site oficial:

https://buildroot.org/downloads/buildroot-2024.02.1.tar.gz

descompacte os arquivos:

	tar xv buildroot-2024.02.1.tar.gz

configure o kernel:

	make menuconfig

compile o kernel:

	make

beleza, essa primeira etapa está pronta.

se entrarmos em output/images, veremos dois arquivos.

* bzImage
* rootfs.tar

### criando ambiente de trabalho

esses arquivos são importantes para a construção da nossa distro.

o bzImage é a imagem do nosso kernel compilado, e o rootfs toda a
arvore de arquivos que será colocada no root e que o kernel precisa
para funcionar adequadamente.

quando eu construi o meu, decidi separá-los em pastas para deixar mais 
organizado, faremos o mesmo aqui.

vamos voltar para o diretório de trabalho onde descompactamos o buildroot
e criar alguns diretórios:

	mkdir image rootfs mounted distro bootImg
	
agora copiamos os arquivos para as respectivas pastas:

	cp buildroot-2024.02.1/output/image/bzImage image
	cp buildroot-2024.02.1/output/image/rootfs.tar rootfs

Explicando rapidamente, o diretório image é o que irá guardar a nossa
bzImage, o rootfs irá guardar o rootfs e o mounted irá ser o ponto
de montagem da nossa imagem inicializável, o diretório distro irá conter
todos os arquivos necessários para construir o sistema, tanto a bzImage
quanto o rootfs, será lá dentro que eles serão unidos e em seguida todos
os arquivos lá dentro serão copiados para dentro de mounted.



certo, agora que organizamos as coisas iremos descompactar o rootfs

	tar xv rootfs/rootfs.tar

vamos mover os arquivos para dentro do diretório distro e copiar
o bzImage

	mv rootfs/rootfs.tar distro
	cp image/bzImage distro

agora iremos criar a imagem inicializável do nosso sistema.

para isso usaremos o comando truncate:

	truncate -s 100MB boot.img

mova ele para bootImg:

	mv boot.img bootImg

certo, o próximo passo é criar o filesystem da imagem, para
que ela possa receber os arquivos.

	mkfs bootImg/boot.img

por fim, tenha certeza de ter o extlinux instalado, que será
o bootloader do nosso linux.

para archlinux:

	sudo pacman -S syslinux

para debian based

	sudo apt install extlinux

agora que temos o bootloader, vamos montar a imagem ao diretório
mounted para copiar os arquivos para lá e por fim instalar o
bootloader.

	sudo mount boot.img mounted
	sudo extlinux --intall mounted
	sudo cp -r distro/* mounted

agora desmonte o diretório.

	sudo umount mounted

pronto! agora você tem uma imagem inicializável do seu sistema!

basta executar isso em uma maquina virtual e fazer o processo de
boot.

quando o boot iniciar, ele irá perguntar como deve proceder.

então digite

/bzImage root=/dev/sda

e você deve ver a tela de login.

parabéns! você acabou de montar um sistema linux do zero :)
