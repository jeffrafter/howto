# Provisioning and CheckInstall

Often in your cookbooks you have recipes that compile packages from source. Doing this for multiple boxes repeatedly can take a long time. For instance, when we cook our box for the first time the process takes a while (maybe 10-20 minutes). 

The `ruby-build` and `node` compilation steps take the longest. This is why tools like Packer exist, so that you can build a base box without needing to recompile everything. If you have a couple of dependencies (ruby, nginx, node) that don't change frequently (or that you have locked to a specific version) you can actually create a package from the source package using [CheckInstall](http://asic-linux.com.mx/~izto/checkinstall/). 

> After you ./configure; make your program, CheckInstall will run make install (or whatever you tell it to run) and keep track of every file modified by this installation, using the excelent installwatch utility written by Pancrazio 'Ezio' de Mauro (p@demauro.net). 

> When make install is done, CheckInstall will create a Slackware, RPM or Debian compatible package and install it with Slackware's installpkg, "rpm -i" or Debian's "dpkg -i" as appropriate, so you can view it's contents with pkgtool ("rpm -ql" for RPM users or "dpkg -l" for Debian) or remove it with removepkg ("rpm -e"|"dpkg -r"). Aditionally, this script will leave you a copy of the installed package in the source directory so you can install it wherever you want, which is my second motivation: I don't have to compile the same software again and again every time I need to install it on another box :-). 

These can be used in a recipe as follows:

    libjpeg = "jpegsrc.v8b.tar.gz"
    libtiff = "tiff-4.0.1.tar.gz"

    dpkg_package "jpeg" do
      case node[:platform]
      when "debian","ubuntu"
        package_name "jpeg"
        source "/var/chef-solo/cookbooks/packages/jpeg_8b-1_amd64.deb"
      end
      action :install
      only_if do
        File.exists?("/var/chef-solo/cookbooks/packages/jpeg_8b-1_amd64.deb")
      end
    end

    dpkg_package "tiff" do
      case node[:platform]
      when "debian","ubuntu"
        package_name "tiff"
        source "/var/chef-solo/cookbooks/packages/tiff_4.0.1-1_amd64.deb"
      end
      action :install
      only_if do
        File.exists?("/var/chef-solo/cookbooks/packages/tiff_4.0.1-1_amd64.deb")
      end
    end

Generally you should not store the large `.deb` packages in your mainline repository. Instead create a new repository and keep them there. You can then download the files using `wget` or `curl` as part of your recipe.