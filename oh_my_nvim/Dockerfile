FROM ubuntu:22.04

ENV user docker
ENV home /home/${user}
ENV nvim_config_repo https://github.com/DerBrunoIR/nvim.git


RUN <<EOF 
	# install utils
	apt update 
	# apt upgrade -y
	apt install -y \
		curl wget sudo man zip unzip tar locales software-properties-common libssl-dev
	
	# enable utf-8 
	echo "LC_ALL=en_US.UTF-8" >> /etc/environment
	echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
	echo "LANG=en_US.UTF-8" > /etc/locale.conf
	locale-gen en_US.UTF-8
EOF

# setup user account
RUN <<EOF 
	# create user
	useradd --create-home --shell /bin/zsh -G root ${user}

	# disable sudo password prompt
	echo "${user}\tALL=(ALL)\tNOPASSWD: ALL" >> /etc/sudoers  
	mkdir -p ~/.local/bin
	export PATH="$PATH:~/.local/bin"
EOF

USER ${user}
WORKDIR ${home}

# setup cli tools
RUN <<EOF
	# yes | sudo unminimize 

	sudo apt install -y \
		tmux zsh git \
		clang make \
		tree xclip bat autojump fd-find ripgrep mawk

	# see 'https://github.com/sharkdp/bat?tab=readme-ov-file#on-ubuntu-using-apt'
	sudo ln -s /usr/bin/batcat /usr/bin/bat  

	# see 'https://github.com/sharkdp/fd?tab=readme-ov-file#on-ubuntu'
	ln -s $(which fdfind) ~/.local/bin/fd

	zsh --version ||  exit 1
	git --version || exit 1
	tree --version || exit 1
	xclip -version || exit 1
	bat --version || exit 1
	autojump --version || exit 1
	fdfind --version || exit 1
	rg --version || exit 1
	awk -W version || exit 1
EOF

# install python
RUN <<EOF
	sudo ln -snf /usr/share/zoneinfo/$(curl https://ipapi.co/timezone) /etc/localtime

	sudo add-apt-repository -y ppa:deadsnakes/ppa
	sudo apt update
	DEBIAN_FRONTEND=noninteractive sudo apt install -y python3.12 python3-pip python3.12-venv
	pip install neovim

	python3.12 --version || exit 1
	pip --version || exit 1
EOF

# install java 
RUN <<EOF
	sudo apt install -y openjdk-21-jdk

	java --version || exit 1
	javac --version || exit 1
EOF

# install luarocks
RUN <<EOF 
	sudo apt install -y lua5.4 luarocks

	lua5.4 -v || exit 1
EOF

# install nodejs, npm
RUN <<EOF
	curl -fsSL https://deb.nodesource.com/setup_21.x | sudo -E bash - && sudo apt install -y nodejs
	sudo npm install -g neovim

	npm --version || exit 1
EOF

# install cargo 
RUN <<EOF
	curl https://sh.rustup.rs -sSf | sh -s -- -y 
	export PATH="$PATH:${home}/.cargo/bin"

	cargo --version || exit 1
EOF

# install go
RUN <<EOF
	 wget -P /tmp https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
	 sudo tar -C /usr/local -xzf /tmp/go1.22.0.linux-amd64.tar.gz
	 export PATH="$PATH:/usr/local/go/bin"
	 go version || exit 1
EOF

# setup cli
RUN <<EOF
	# install ohmyzsh
	yes | sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

	# install starship prompt
	mkdir -p ~/.local/bin ~/.config
	curl -sS https://starship.rs/install.sh | sh -s -- -f -b ~/.local/bin
	export PATH="$PATH:${home}/.local/bin"
	starship --version || exit 1
EOF

# setup nvim
RUN <<EOF
	sudo apt install pkg-config
	# install nvim from source
	curl -L https://github.com/neovim/neovim/releases/latest/download/nvim-linux64.tar.gz -o /tmp/nvim-linux64.tar.gz
	sudo tar -C /opt -xzf /tmp/nvim-linux64.tar.gz
	rm /tmp/nvim-linux64.tar.gz
	export PATH="$PATH:/opt/nvim-linux64/bin"

	# install configuration
	git clone --depth=1 ${nvim_config_repo} ~/.config/nvim

	# install vim plug
	sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

	# install and init all plugins
	# on startup nvim throws lots of errors because all configured plugins are missing
	nvim --headless +PlugInstall +qall || exit 1
	nvim --headless +TSUpdate +qall || exit 1
	nvim --headless +MasonUpdate +MasonInstall +qall || exit 1
EOF


COPY ./dotfiles/* ${home}/
COPY ./starship.toml ${home}/.config/starship.toml

# clear /tmp
RUN sudo rm -rf /tmp/*

CMD [ "zsh" ]

