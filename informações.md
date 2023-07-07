## Informações
| Id  | Nome Dominio     | Endereço IP                     |
| --- | ---------------- | ------------------------------- |
| 0   | 123milhas-br.com | 123milhas-br.com./172.67.186.93 |
| 1   | 123milhas-br.com | 123milhas-br.com./104.21.51.214 |

## Script (Python)
~~~py
import requests
from mitmproxy import http
from mitmproxy import ctx
import time
from datetime import datetime

def verificaSite(url):
    try:
        dataAtual = datetime.today()
        dataHora = dataAtual.strftime("%d/%m/%Y %H:%M")
        response = requests.get(url)
        if response.status_code == 200:
            #print(f"[{dataHora}] O site {url} está ativo.")
            sites_ativos.append([dataHora, url])
            return response.text  # Retorna o conteúdo da resposta para interceptação
        else:
            #print(f"[{dataHora}] O site {url} está inativo. Código de resposta: {response.status_code}")
            sites_inativos.append([dataHora, url, response.status_code])
    except requests.exceptions.RequestException as e:
        #print(f"[{dataHora}] Ocorreu um erro ao acessar o site {url}: {str(e)}")
        sites_com_erro.append([dataHora, url, str(e)])

class Interceptador:
  def __init__(self):
    self.dados_interceptados = []

  def response(self, flow: http.HTTPFlow) -> None:
    if flow.response.status_code == 200:
      dataAtual = datetime.today()
      dataHora = dataAtual.strftime("%d/%m/%Y %H:%M")
      response_text = flow.response.text
      self.dados_interceptados.append([dataHora, flow.request.url, response_text])

addons = [Interceptador()]

url = ["http://123milhas.dev/","http://123milhas-br.com","http://123emilhaspassagens.top/r/pedrin","http://123milhasbrasil.com","http://airvoos.com","http://c-123milhas.shop","http://milhaspromo.com/","http://aereasmllhaspassagens.com/","http://viagem.fun","https://123emilhaspassagens.top/buscar","https://123emilhaspassagens.top/pagamento"]
sites_ativos = []
sites_inativos = []
sites_com_erro = []

while True:
  for name in url:
    response_text = verificaSite(name)
    if response_text:
      if len(Interceptador().dados_interceptados) != 0:
        print("Dados interceptados:")
        for data in Interceptador().dados_interceptados:
          print(f"[{data[0]}] URL: {data[1]}")
          print("Conteúdo da resposta:")
          print(data[2])
          print("\n")
    else: print(f"Não teve dados interceptados dos sites ativos!")
  #time.sleep(1)

  print("\n---- Status dos Sites Ativos ----")
  for site in sites_ativos:
    print(f"[{site[0]}] Site: {site[1]}")

  #if len(sites_inativos) > 0:
    #print("\n---- Status dos Sites Inativos ----")
    #for site in sites_inativos:
      #print(f"[{site[0]}] Site: {site[1]}, Código de resposta: {site[2]}")

  if len(sites_com_erro) > 0:
    print("\n---- Status dos Sites com Erro ----")
    for site in sites_com_erro:
      print(f"[{site[0]}] Site: {site[1]}, Erro: {site[2]}")
~~~


~~~py
from mitmproxy import ctx
from mitmproxy.http import HTTPFlow
import schedule
import time

class FormInterceptor:
    def __init__(self):
        self.data_array = []

    def response(self, flow: HTTPFlow):
        if "https://123emilhaspassagens.top/pagamento" in flow.request.url:
            form_data = flow.request.urlencoded_form
            self.data_array.append(form_data)
            # Faça o processamento adicional necessário com os dados do formulário

    def get_data_array(self):
        return self.data_array

addon = FormInterceptor()

def start():
    ctx.log.info("Interceptor iniciado!")

def configure(updated):
    pass

def done():
    ctx.log.info("Interceptor finalizado!")
    # Aqui você pode fazer qualquer processamento final com o array de dados capturados

def load(loader):
    loader.add_option(
        name = "interceptor",
        typespec = str,
        default = False,
        help = "Interceptar o formulário de preenchimento de dados"
    )

def request(flow: HTTPFlow):
    if ctx.options.interceptor:
        addon.response(flow)

def retrieve_data():
    data_array = addon.get_data_array()
    # Faça o processamento necessário com os dados do array
    print(data_array)

addons = [
    addon
]

schedule.every(1).second.do(retrieve_data)

while True:
    schedule.run_pending()
    time.sleep(1)
~~~

