# Use the empty image with the portage tree as the first stage
FROM gentoo/portage:latest as portage

# Gentoo stage3 is the second stage, basically an unpacked Gentoo Linux
FROM gentoo/stage3-amd64:latest as gentoo

# Copy the portage tree into the current stage
COPY --from=portage /usr/portage /usr/portage

MAINTAINER alicef@gentoo.org

RUN eselect profile set "default/linux/amd64/17.0/hardened"
RUN rm /var/lib/portage/world && rm -R /etc/portage/package.* /etc/portage/make.conf
RUN wget https://gitgud.io/cloveros/cloveros/raw/master/binhost_settings/etc/portage/{make.conf,package.use,package.keywords,package.env,package.mask,package.unmask} -P /etc/portage/ && mkdir /etc/portage/env && wget https://gitgud.io/cloveros/cloveros/raw/master/binhost_settings/etc/portage/env/{gold,no-gnu2,no-gold,no-hashgnu,no-lto,no-lto-graphite,no-lto-graphite-ofast,no-lto-o3,no-lto-ofast,no-ofast,pcsx2,size} -P /etc/portage/env/ && wget https://gitgud.io/cloveros/cloveros/raw/master/binhost_settings/var/lib/portage/world -O /var/lib/portage/world

RUN CFLAGS="-Ofast -mmmx -mssse3 -pipe -flto=8 -funroll-loops" emerge gcc

RUN binutils-config --linker ld.gold

RUN USE="-vaapi" emerge -1 mesa
RUN USE="binary" emerge -1 scala
RUN emerge -1 netcat6
RUN emerge genkernel gentoo-sources
RUN wget https://liquorix.net/sources/4.19/config.amd64
RUN binutils-config --linker ld.bfd ; genkernel --kernel-config=config.amd64 all ; binutils-config --linker ld.gold
RUN emerge layman
RUN layman -S
RUN yes | layman -a $(grep -Po "(?<=\*/\*::).*" /etc/portage/package.mask | tr "\n" " ")
RUN emerge -uvDN @world
RUN emerge -C hwinfo ntfs3g && emerge ntfs3g && emerge hwinfo
RUN quickpkg --include-unmodified-config=y "*/*"
