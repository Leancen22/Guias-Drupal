#  Deploy en servidores de Inversor y Museos

Una vez conectado a la VPN.

Moverse hasta la carpeta correspondiente

      cd ../..
      cd inversor o cd museos
      git pull
      ./vendor/bin/drush cim --yes
      ./vendor/bin/drush cr
