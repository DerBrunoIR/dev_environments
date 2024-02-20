FROM ubuntu:22.04

ENV user docker
ENV home /home/${user}
ENV nvim_config_repo https://github.com/DerBrunoIR/nvim.git

# enable utf-8 
ENV LC_ALL=en_US.UTF-8
ENV LANG=en_US.UTF-8

# install utils
RUN <<EOF 
	apt update 
	# apt upgrade -y
	apt install -y \
		curl wget sudo man zip unzip tar
EOF

# setup user account
RUN <<EOF 
	# create user
	useradd --create-home --shell /bin/zsh -G root ${user}

	# disable sudo password prompt
	echo "${user}\tALL=(ALL)\tNOPASSWD: ALL" >> /etc/sudoers  
EOF

USER ${user}
WORKDIR ${home}

# setup cli tools
RUN <<EOF
	yes | sudo unminimize 

	sudo apt install -y \
		zsh git \
		tree xclip bat autojump

	# see 'https://github.com/sharkdp/bat?tab=readme-ov-file#on-ubuntu-using-apt'
	sudo ln -s /usr/bin/batcat /usr/bin/bat  

	zsh --version ||  exit 1
	git --version || exit 1
	tree --version || exit 1
	xclip -version || exit 1
	bat --version || exit 1
	autojump --version || exit 1
EOF

# setup cli
RUN <<EOF
	# install ohmyzsh
	yes | sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

	# install starship prompt
	mkdir -p ~/.local/bin ~/.config
	curl -sS https://starship.rs/install.sh | sh -s -- -f -b ~/.local/bin
	export PATH=$PATH:${home}/.local/bin
	starship --version || exit 1
EOF

# setup nvim
RUN <<EOF
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
	nvim --headless +MasonUpdate +qall || exit 1
EOF

COPY ./zshrc ${home}/.zshrc
COPY ./starship.toml ${home}/.config/starship.toml
COPY ./install_lsp_dependencies.sh ${home}/install_lsp_dependencies

CMD [ "zsh" ]

