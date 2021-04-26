---

title: "Simula��o do problema de Monty Hall em R"

categories: []

date: '2021-04-25T00:00:00Z'

draft: no

featured: no

gallery_item: null

image:
  caption: 
  focal_point: Top
  preview_only: no

projects: []

subtitle: null

summary: null

tags:
- Reshape 
- Pivot Wider
- Pivot Longer
- Research

authors:
- GabrielBoechat


---

## Comandos Gerais

O problema de Monty Hall surgiu e foi nomeado pelo nome do apresentador de um programa de televis�o dos anos 70, nos EUA, similar com o que vemos no S�lvio Santos aqui no Brasil. Pelo nome pode n�o lembrar, mas deve lembrar pela cena do excelente filme ["Quebrando a Banca"](https://www.youtube.com/watch?v=B6kYbt4LyLA). (caso n�o tenha visto, recomendamos bastante!)

Voc� tem 3 portas na sua frente: uma com um carro e outras duas com bodes, apenas n�o sabe quais s�o. Logo ap�s voc� escolher uma porta, Monty, que sabe qual tem o carro, abre uma com um bode por tr�s, e pergunta: "Voc� gostaria de manter ou trocar a porta escolhida?"

Algo parece estranho... Se voc� troca a porta, existe 50% de chance de ter um bode ou de ter um carro, torna-se aleat�rio, certo? Na verdade, n�o. Como voc� escolheu uma porta ao acaso, h� maior chance de ter inicialmente escolhido com um bode atr�s (2 possibilidades em 3, ou 2/3 = 67%) e, como Monty Hall mostra a porta que tem um bode, voc� tem mais chance de trocar para uma que realmente tenha o pr�mio. 

Assim, a estrat�gia de manter a porta te d� 1/3 de chance de acertar, enquanto trocando suas chances dobram, indo para 2/3!

Como podemos achar essas probabilidades simulando v�rios jogos? � isso que vamos explorar a seguir:

Iniciando as vari�veis do jogo

    library(tidyverse) # Pacote para limpeza e visualiza��o de dados
    library(grid)      # Nos auxiliar� para fazer anota��es nos gr�ficos
    library(ggpubr)    # Temas j� personalizados
    
    set.seed(1970) # Homenagem � d�cada que o programa foi ao ar
                   # Ter o mesmo valor permite que tenha os mesmos resultados do c�digo abaixo
    
    n <- 5000 # N�mero de programas que simularemos
    
    resultados <- matrix(data = NA, # Matriz vazia; usaremos para preenchermos os resultados 
                         ncol = 3,  # Tr�s colunas: n�mero do jogo e se acerta mantendo ou trocando a porta
                         nrow = n)  # Quantidade de jogos, um em cada linha
                        
    resultados[,1] <- 1:n # Enumerando os jogos, de 1 at� n (5000 nesse caso)                    

Modelando o desenrolar do jogo

      
    for(i in c(1:n)) {
      
      portas = rep(NA, 3) # Novas portas
      
      portas[sample(1:3, 1)] = 1 # Apenas uma delas cont�m o pr�mio, designada ao acaso
      
      portas[is.na(portas) == TRUE] = 0 # Quais n�o cont�m pr�mio s�o aquelas que tem bodes
      
      jogador = sample(1:3, 1) # Jogador escolhe uma das 3 portas ao acaso
      
      jogo = matrix(data = c( portas, c(1:3) ), nrow = 2, byrow = TRUE) # Apenas juntando as informa��es do jogo at� aqui
      
      jogo_apresentador = jogo[1,] # O apresentador, por�m, conhece o jogo e qual a porta vencedora
      
      jogo_apresentador[jogador] = "N�O USAR"  # Obviamente, o apresentador n�o abrir� a porta que o jogador tenha escolhido
      
      reveal = which(jogo_apresentador %in% "0") # Sabendo que existe outra porta com o bode, o apresentador a escolhe
      
      if(length(reveal) == 2) {
        
        reveal = sample(reveal, 1)
        
      }
      

Arrumando os resultados para visualizarmos graficamente

    df_resultados <- as.data.frame(resultados) # Transformando em data.frame

    names(df_resultados) <- c("Itera��o", "Mant�m", "Troca") # Modificando os nomes

    df_resultados$Mant�m <- cummean(df_resultados$Mant�m) # Observando a m�dia de acertos ao longo dos jogos 
    df_resultados$Troca <- cummean(df_resultados$Troca)

    df_resultados_gather <- gather(df_resultados,       # Organizando os dados num formato longo
                                   key = "Estrat�gia",  # Colocar "Mant�m" e "Troca" numa coluna apenas
                                   value = "Acertos",
                                   -Itera��o)
                            
Visualiza��o gr�fica (utilizando ggplot2)

    ggplot(data = df_resultados_gather, mapping = aes(x = Itera��o,           # N�mero do jogo em X
                                                      y = Acertos,            # Valor da probabilidade de acerto em Y
                                                      colour = Estrat�gia)) + # Colorindo as curvas por tipo de estrat�gia
    geom_line() + # Fazendo o gr�fico das curvas
    geom_hline(yintercept = 1/3,
               lty = 2,
               lwd = 0.5) + # Colocando no gr�fico a probabilidade verdadeira de ganhar n�o trocando
    geom_hline(yintercept = 2/3,
               lty = 2,
               lwd = 0.5) + # Colocando no gr�fico a probabilidade verdadeira de ganhar trocando
    geom_text(data = subset(df_resultados_gather, Itera��o == n),
            aes(label = Estrat�gia, 
                colour = Estrat�gia, 
                x = n, 
                y = Acertos + 0.05)) + # Marcando no gr�fico os nomes das curvas
    ggpubr::theme_classic2() + # Tema j� pronto para uso
    scale_y_continuous(labels = scales::percent_format()) + # Colocando o eixo Y em percentual
    labs(x = NULL,
         y = "Probabilidade de vencer",
         title = "Problema de Monty Hall") + # Trocando os t�tulos dos eixos
    theme(legend.position = "none") + # Retirando a legenda
    scale_color_manual(values = c("green", "red")) # Mudando manualmente as cores das curvas