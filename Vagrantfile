# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
	:packager => {
		:box_name => "centos/stream8",
		:box_version => "20210210.0",
		:ip_addr => "192.168.11.11"
	}
}

Vagrant.configure("2") do |config|
	MACHINES.each do |boxname, boxconfig|
        config.vm.box_version= boxconfig[:box_version]
		config.vm.define boxname do |box|
			box.vm.box = boxconfig[:box_name]
			box.vm.hostname = boxname.to_s
			box.vm.network "private_network", ip: boxconfig[:ip_addr]
			if boxname.to_s == "packager" then
				box.vm.provision "shell", inline: <<-'SHELL'
                yum install -y -q redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils gcc perl-IPC-Cmd
                echo "download nginx and openssl"
                wget -nv https://nginx.org/packages/centos/8/SRPMS/nginx-1.22.1-1.el8.ngx.src.rpm
                wget -nv https://github.com/openssl/openssl/releases/download/openssl-3.0.8/openssl-3.0.8.tar.gz
                echo "extract and install sources"
                tar -xf openssl-3.0.8.tar.gz -C /root/
                rpm -i nginx-1.22.1-1.el8.ngx.src.rpm --quiet
                echo "install required packages"
                yum-builddep /root/rpmbuild/SPECS/nginx.spec -y --quiet
                sed -i 's/--with-debug/--with-openssl=\/root\/openssl-3.0.8 \\\n   --with-debug/' /root/rpmbuild/SPECS/nginx.spec
                echo "build RPMs"
                rpmbuild -bb /root/rpmbuild/SPECS/nginx.spec --quiet
                echo "check built RPMs"
                ls -l /root/rpmbuild/RPMS/x86_64/
                mkdir /usr/share/localrepo
                cp /root/rpmbuild/RPMS/x86_64/nginx-* /usr/share/localrepo/
                createrepo /usr/share/localrepo/
                echo "installing docker"
                yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
                yum install -qy docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
                echo -e "server {\n\tlisten\t80;\n\tserver_name\tlocalhost;\n\tlocation\t/ {\n\t\troot\t/usr/share/nginx/html;\n\t\tautoindex on;\n\t}\n}" > /root/autoindex.conf
                echo -e "[local-http]\nname=Repo on docker\nbaseurl=http://localhost:8080\ngpgcheck=0\nenabled=1" > /etc/yum.repos.d/local-http.repo
                echo -e "[local-folder]\nname=Repo in localfile\nbaseurl=file:///usr/share/localrepo\ngpgcheck=0\nenabled=1" > /etc/yum.repos.d/local-folder.repo
                systemctl start docker
                echo "start nginx container with repo"
                docker run --quiet -it --rm -d -p 8080:80 --name web -v /usr/share/localrepo/:/usr/share/nginx/html -v /root/autoindex.conf:/etc/nginx/conf.d/default.conf nginx
                echo "check local-http repo contents"
                yum --disablerepo="*" --enablerepo="local-http" list available
                echo "check local-folder repo contents"
                yum --disablerepo="*" --enablerepo="local-folder" list available
				SHELL
			end
		end
	end
end
