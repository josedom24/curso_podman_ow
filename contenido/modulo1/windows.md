# Instalación de Podman en Windows/macOS

Para el sistema operativo macOs puedes encontrar información en la página oficial de [instalación de Podman](https://podman.io/docs/installation#macos).

En la [documentación oficial](https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md) sobre la instalación de Podman en Windows puedes encontrar la información completa.

En ambos sistemas Podman creará una máquina virtual Linux donde se ejecura Podman y se accederá desde el cliente que lo tendremos en el sistema anfitrión.

## Instalación en Windows

Los requistos para realizar la instalción son los siguientes:

* La máquina virtual se crea en WSL (El subsistema Windows para Linux).
* Necesitamos una versión reciente de Windows 10 o Windows 11. Se requiere la compilación 18362 o superior.
* Su sistema debe soportar y tener habilitada la virtualización de hardware. 

1. Descagamos el cliente de Podman desde la página de descargas [GitHub release page](https://github.com/containers/podman/releases). En este ejemplo vamos a bajar la versión 5.0.2, el fichero `podman-5.0.2-setup.exe`.
2. Lo ejecutamos y realizamos la instalación.
    ![win](img/win1.png)
3. Abrimos una terminal de PowerShell y comenzamos el proceso de creación de la máquina virtual:

    ```
    > podman machine init
    ```

    ![win](img/win2.png)

4. Podemos ver las máquinas virtuales que hemos creado ejecutando:

    ```
    > podman machine list
    ```

    Inicamos la máquina virtual:

    ```
    > podan machine start
    ```
    Por edefecto la máquina virtual se ha inciado para trabajar con contenedores rootless. Si queremos usar contenedores rootful, debemos ejecutar con la máquina parada:

    ```
    > podman machine set --rotful
    ```
    ![win](img/win3.png)

5. Ya podemos utilziar Podman:

    ![win](img/win4.png)

6. Otras operaciones que podemos hacer con la máquina virtual. Si queremos acceder por ssh a la máquina virtual:

    ```
    > podman machine ssh
    ```

    Para para la máquina virtual:

    ```
    > podman machine stop
    ```

    Para eliminar la máquina virtual:

    ```
    > podman machine rm
    ```
