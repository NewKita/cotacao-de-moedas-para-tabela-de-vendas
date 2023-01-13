from selenium import webdriver
from selenium.webdriver.common.keys import Keys

navegador = webdriver.Chrome()

#Passo 1 - Pegando cotação dólar
navegador.get("https://www.google.com.br/")
# Digitando a pesquisa
navegador.find_element('xpath', '/html/body/div[1]/div[3]/form/div[1]/div[1]/div[1]/div/div[2]/input').send_keys("Cotação dólar")
# Iniciando a pesquisa
navegador.find_element('xpath', '/html/body/div[1]/div[3]/form/div[1]/div[1]/div[1]/div/div[2]/input').send_keys(Keys.ENTER)
# Selecionando o valor da pesquisa
cotacao_dolar = navegador.find_element('xpath', '//*[@id="knowledge-currency__updatable-data-column"]/div[1]/div[2]/span[1]').get_attribute('data-value')

print(cotacao_dolar)

#Passo 2 - Pegando cotação euro - mesma pesquisa do dólar, só mudando o nome
navegador.get("https://www.google.com.br/")
navegador.find_element('xpath', '/html/body/div[1]/div[3]/form/div[1]/div[1]/div[1]/div/div[2]/input').send_keys("Cotação euro")
navegador.find_element('xpath', '/html/body/div[1]/div[3]/form/div[1]/div[1]/div[1]/div/div[2]/input').send_keys(Keys.ENTER)
cotacao_euro = navegador.find_element('xpath', '//*[@id="knowledge-currency__updatable-data-column"]/div[1]/div[2]/span[1]').get_attribute('data-value')

print(cotacao_euro)

#Passo 3 - Pegando cotação ouro
navegador.get("https://www.melhorcambio.com/ouro-hoje")
cotacao_ouro = navegador.find_element('xpath', '//*[@id="comercial"]').get_attribute('value')
cotacao_ouro = cotacao_ouro.replace(",", ".")
print(cotacao_ouro)

navegador.quit() # No final da pesquisa ele fecha o navegador.

# Passo 4 - Importar a base de dados
import pandas as pd

tabela = pd.read_excel(r"Local do arquivo")
display(tabela)

# Passo 5 - Recalcular os preços
# Atualizar cotação
# Cotação dolar

tabela.loc[tabela["Moeda"] == "Dólar", "Cotação"] = float(cotacao_dolar)
tabela.loc[tabela["Moeda"] == "Euro", "Cotação"] = float(cotacao_euro)
tabela.loc[tabela["Moeda"] == "Ouro", "Cotação"] = float(cotacao_ouro)

# Preço de compra = cotação * preço original
tabela["Preço de Compra"] = tabela["Cotação"] * tabela["Preço Original"]

# Preço da venda = Preço da compra * margem
tabela["Preço de Venda"] = tabela["Preço de Compra"] * tabela["Margem"]

tabela["Preço de Venda"] = tabela["Preço de Venda"].map("R$ {:.2f}".format)
tabela["Preço de Compra"] = tabela["Preço de Compra"].map("R$ {:.2f}".format)

display(tabela)

# Passo 6 - Exportar a base atualizada

tabela.to_excel("Produtos Novos.xlsx", index=False)
