esta no esta, ademas quiero todo el vps q se acceda de code server, porque si estoy desarrollando ejemplo el repro del vps root/.hermes debe ser como si code server estuviera ejecutandose como un proceso mas en el vps, dicho de otra manera usar codeserver desarrolladocon vscode server remote o codeserver debe ser transparentepara mi 

.hermes es un ejemplo no te centres en el 

/home/coder/workspaces esta .hermes

pero si pongo en open folders /vps no encuentra nada
ejemplo practico

root@0c8fe0ef268f:~/workspaces/hermes-web-ui# st
bash: st: command not found
root@0c8fe0ef268f:~/workspaces/hermes-web-ui# ./start.sh 
==========================================
  Hermes Agent Web UI
    http://127.0.0.1:5000
    ==========================================

      Python runtime not found at /root/workspaces/hermes-web-ui/venv/bin/python
        Set WEBUI_VENV to the Hermes/web UI virtualenv and try again.
        ==========================================
        root@0c8fe0ef268f:~/workspaces/hermes-web-ui# 

al estar dentro del contenedor:
1 no encuentra la variable de entorno del vps
2. las rutas no son iguales anade ~/workspaces cuando si estoy con vscode server es ~/hermes-web-ui# 

conclusion

hay que ver si se puede instalar code-server directamente en el vps con usuario root sin que afecte a vscode server