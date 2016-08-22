# A dummy plugin for Barge to set hostname and network correctly at the very first `vagrant up`
module VagrantPlugins
  module GuestLinux
    class Plugin < Vagrant.plugin("2")
      guest_capability("linux", "change_host_name") { Cap::ChangeHostName }
      guest_capability("linux", "configure_networks") { Cap::ConfigureNetworks }
    end
  end
end

GO_VERSION = "1.7"

Vagrant.configure(2) do |config|
  config.vm.define "clair-barge"

  config.vm.box = "ailispaw/barge"

  config.vm.network :private_network, ip: "192.168.33.10"

  config.vm.synced_folder ".", "/vagrant", type: "nfs",
    mount_options: ["nolock", "vers=3", "udp", "noatime", "actimeo=1"]

  config.vm.provision :shell, run: "always" do |sh|
    sh.inline = <<-EOT
      pkg install bindfs

      if mountpoint -q /vagrant && ! mountpoint -q /opt/data; then
        mkdir -p /vagrant/data
        mkdir -p /opt/data
        _UID=$(ls -ld /vagrant | awk '{print $3}')
        _GID=$(ls -ld /vagrant | awk '{print $4}')
        bindfs --map=${_UID}/999:@${_GID}/@999 /vagrant/data /opt/data
      fi

      mkdir -p /opt/tmp
      chmod a+w /opt/tmp
    EOT
  end

  config.vm.provision :shell, run: "always" do |sh|
    sh.inline = <<-EOT
      docker stop postgres 2>/dev/null || true
      docker rm -f postgres 2>/dev/null || true
      docker stop clair 2>/dev/null || true
      docker rm -f clair 2>/dev/null || true
    EOT
  end

  config.vm.provision "postgres", type: "docker", run: "always" do |docker|
    docker.pull_images "postgres:9.5"
    docker.run "postgres",
      image: "postgres:9.5",
      args: [
        "-e POSTGRES_PASSWORD=test",
        "-v /opt/data:/var/lib/postgresql/data"
      ].join(" "),
      restart: false
  end

  config.vm.provision "clair", type: "docker", run: "always" do |docker|
    docker.pull_images "quay.io/coreos/clair:v1.2.2"
    docker.run "clair",
      image: "quay.io/coreos/clair:v1.2.2",
      args: [
        "-p 6060-6061:6060-6061",
        "-v /vagrant/config:/config",
        "-v /opt/tmp:/opt/tmp",
        "--link postgres:postgres"
      ].join(" "),
      cmd: "-config=/config/config.yaml",
      restart: false
  end

  config.vm.provision :shell do |sh|
    sh.privileged = false
    sh.inline = <<-EOT
      if [ ! -d /opt/go ]; then
        echo "Installing go v#{GO_VERSION}"
        cd /opt
        wget -qO- https://storage.googleapis.com/golang/go#{GO_VERSION}.linux-amd64.tar.gz | sudo tar xz
        echo "export GOROOT=/opt/go" >> ~/.bash_profile
        echo "export GOPATH=\\$HOME/go" >> ~/.bash_profile
        echo "export PATH=$PATH:\\$GOROOT/bin:\\$GOPATH/bin" >> ~/.bash_profile
        echo "export TMPDIR=/opt/tmp" >> ~/.bash_profile
        source ~/.bash_profile
      fi

      sudo pkg install git
      git config --global http.sslCAinfo /etc/ssl/certs/ca-certificates.crt
      go get -u github.com/coreos/clair/contrib/analyze-local-images
    EOT
  end
end
