FROM ubuntu:22.04

ENV user bruno
ENV home /home/${user}
ENV nvim https://github.com/DerBrunoIR/nvim.git

RUN yes | unminimize 

# setup enviroment
RUN <<EOF
	apt update 
	apt upgrade -y

	# cli utils
	apt install -y \
		zsh git tmux wget curl \
		tree xclip bat autojump \
		tar zip unzip \
		gcc make 

	# see 'https://github.com/sharkdp/bat?tab=readme-ov-file#on-ubuntu-using-apt'
	ln -s /usr/bin/batcat /usr/bin/bat  

	# setup user
	useradd --create-home --shell /bin/zsh -G root ${user}

	# install neovim from source
	curl -L https://github.com/neovim/neovim/releases/latest/download/nvim-linux64.tar.gz -o /tmp/nvim-linux64.tar.gz
	tar -C /opt -xzf /tmp/nvim-linux64.tar.gz
	rm -rf /tmp/nvim-linux64.tar.gz

	# clear tmp
	rm -rf /tmp/*
EOF

USER ${user}
WORKDIR ${home}

RUN <<EOF
	# install oh my zsh
	yes | sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

	# install starship prompt
	mkdir -p ~/.local/bin ~/.config
	curl -sS https://starship.rs/install.sh | sh -s -- -f -b ~/.local/bin

	# setup neovim configuration
	git clone --depth=1 ${nvim} ~/.config/nvim

	# install vim plug
	sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

	# install all neovim plugins
	export PATH="$PATH:/opt/nvim-linux64/bin"
	nvim --headless +PlugInstall +TSUpdate +MasonInstall +checkhealth +qall
EOF

COPY ./zshrc ${home}/.zshrc
COPY ./starship.toml ${home}/.config/starship.toml

CMD [ "zsh" ]

