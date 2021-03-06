

## Gráficos  

### Gramática de gráficos

Existem várias abordagens para criar gráficos no R e o foco aquí é a *gramática de gráficos* implementada no pacote ggplot2. A ideia da gramática dos gráficos é usar geometrias com propriedades estéticas para representar dados em um sistema de coordenadas. Para entendermos melhor a gramática de gráficos, criemos um banco fictício de casos notificados e confirmados durante 200 semanas, com o sexo e o local de ocorrência de cada caso.


```r
> set.seed(12)
> casos <- data.frame(
+   semana = 1:200,
+   notificados = round(seq(50, 250, l = 200) + runif(200, -40, 40)),
+   confirmados = round(seq(5, 25, l = 200) + runif(200, -5, 5)),
+   sexo = c(sample(c('M', 'F'), 100, replace = T),
+            sample(c('M', 'F'), 100, replace = T, prob = c(.8, .2))),
+   local = rep(c('A', 'B'), each = 50))
```


```r
> str(casos, v = 1)
```

```
'data.frame':	200 obs. of  5 variables:
 $ semana     : int  1 2 ...
 $ notificados: num  16 76 ...
 $ confirmados: num  10 2 ...
 $ sexo       : Factor w/ 2 levels "F","M": 2 2 ...
 $ local      : Factor w/ 2 levels "A","B": 1 1 ...
```

```r
> head(casos)
```

```
  semana notificados confirmados sexo local
1      1          16          10    M     A
2      2          76           2    M     A
3      3          87           5    M     A
4      4          35           9    F     A
5      5          28          10    F     A
6      6          18           7    M     A
```

A partir do banco `casos` podemos graficar a relação entre o número de casos notificados e confirmados usando a função `ggplot`. O primeiro passo é a especificação do banco de dados no argumento `data`, e das variáveis a serem representadas no argumento `mapping`. Este último argumento é definido com a função `aes` que permite especificar as propriedades estéticas que mapearão (representarão) as variáveis.  


```r
> library(ggplot2)
```

```
Find out what's changed in ggplot2 at
http://github.com/tidyverse/ggplot2/releases.
```

```r
> ggplot(data = casos, mapping = aes(x = notificados, y = confirmados))
```

![plot of chunk 06-01](figures/06-01-1.png)

No gráfico acima, as variáveis `notificados` e `confirmados` foram mapeadas nos eixos `x` e `y` que são duas propriedades estéticas. Entretanto, o valor dessas variáveis para cada observação (semana) não foi graficado porque não especificamos nenhuma geometria. Se o objetivo é criar um gráfico de dispersão (scatterplot) podemos acrescentar a função `geom_point` (reparem no sinal + que acrescenta a função) para usar o ponto como geometria.  


```r
> ggplot(data = casos, mapping = aes(notificados, confirmados)) +
+   geom_point()
```

![plot of chunk 06-02](figures/06-02-1.png)

Em capítulos anteriores usamos a função `plot` - que não segue os princípios da gramática de gráficos - e aqui podemos usá-la de novo para criar um gráfico equivalente ao anterior.  


```r
> plot(casos$notificados, casos$confirmados)
```

![plot of chunk 06-03](figures/06-03-1.png)

Por que usar a função `ggplot` se a função `plot` é muito mais simples? Porque é comum criar gráficos com mais informações (legendas, relações multivariadas, etc.) e nesses casos a função `ggplot` é mais versátil e fácil de usar. Por exemplo, para representar o sexo com cores diferentes e criar a legeneda correspondente, só precisamos mapear a variável `sexo` na propriedade estética `color`.


```r
> ggplot(data = casos, aes(notificados, confirmados, color = sexo)) +
+   geom_point()
```

![plot of chunk 06-04](figures/06-04-1.png)

Embora seja possível criar um gráfico equivalente com a função `plot`, o nível de dificuldade é maior. E conforme aumenta a complexidade dos gráficos, maior a diferença entre as abordagens. Por exemplo, como a gramática de gráficos é simples criar um gráfico da relação entre o o número de casos notificados e confirmados, condicionada no local de ocorrência e o sexo.  


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_grid(local ~ sexo)
```

![plot of chunk 06-05](figures/06-05-1.png)

Criar um gráfico equivalente ao anterior com as funções básicas do R seria um bom exercício para quem está entediado e quer reproduzir com muitas linhas de código o que dá para fazer com poucas. Cabe mencionar o lattice que um pacote básico do R e implementa uma outra abordagem também apropriada para gráficos de diversa complexidade. Porém, o foco aqui é o ggplot2.  

#### Mapeamento de variáveis em propriedades estéticas

Uma das principais fontes de confusão no uso do ggplot2 refere-se à forma de usar as propriedades estéticas. Uma propriedade estética pode ser definida como igual a um determinado valor ou mapear uma variável. Por exemplo, a propriedade estética `color` pode ser definida como igual a `'blue'`


```r
> ggplot(data = casos, mapping = aes(notificados, confirmados)) +
+   geom_point(color = 'blue')
```

![plot of chunk 06-06](figures/06-06-1.png)

ou mapear a variável `sexo` como feito anteriormente.


```r
> ggplot(data = casos, aes(notificados, confirmados, color = sexo)) +
+   geom_point()
```

![plot of chunk 06-07](figures/06-07-1.png)


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point(aes(color = sexo))
```

![plot of chunk 06-08](figures/06-08-1.png)

A propriedade estética sempre mapeará uma variável quando usada dentro da função `aes`, que pode ir tanto na função `ggplot` como nas funções com prefixo `geom_ `.

Ao usar `color = 'blue'` dentro da função `aes`, `blue` passa a ser uma variável que por não existir no banco, gera resultados sem sentido.  


```r
> ggplot(data = casos, aes(notificados, confirmados, color = 'blue')) +
+   geom_point()
```

![plot of chunk 06-09](figures/06-09-1.png)

Reiterando, a propriedade estética deve ir dentro da função `aes` para mapear variáveis, e nas funções com prefixo `geom_` (e fora da função `aes`) para defini-las como igual a um determinado valor.

### Gráficos de uma variável quantitativa

#### Histograma

Em um histograma, a variável contínua é dividida em intervalos e a quantidade de observações em cada intervalo é representada pela altura de uma barra.  


```r
> his <- ggplot(casos, aes(notificados))
> his + geom_histogram()
```

```
`stat_bin()` using `bins = 30`. Pick better value with
`binwidth`.
```

![plot of chunk 06-10](figures/06-10-1.png)

A mensagem gerada pelo comando acima indica que por padrão foi usado `bins = 30` para criar 30 intervalos. Por tanto, podemos controlar o número de intervalos com o argumento `bins`.  


```r
> his + geom_histogram(bins = 10)
```

![plot of chunk 06-11](figures/06-11-1.png)

Também podemos controlar a amplitude dos intervalos.  


```r
> his + geom_histogram(binwidth = 5)
```

![plot of chunk 06-12](figures/06-12-1.png)

A cor de prenchimento é definida pelo argumento `fill` e a cor de contorno pelo argumento `color`.


```r
> his + geom_histogram(bins = 10, fill = 'white', color = 'green')
```

![plot of chunk 06-13](figures/06-13-1.png)

Para representar a contribuição relativa de cada espécie em cada intervalo, o argumento `fill` deve ser usado como propriedade estética para mapear a variável `Species`. Em outras palavras, `fill` deve ser usado como argumento da função `aes`, e não como argumento de `geom_histogram`.  


```r
> his <- ggplot(casos, aes(notificados))
> his + geom_histogram(bins = 10)
```

![plot of chunk 06-14](figures/06-14-1.png)

#### Densidade

Os gráficos de densidade representam a densidade probabilística de uma variável quantitativa. A área sob a curva é uma medida de probabilidade e assim, ao dividir a variável em intervalos equidistantes, a probabilidade associada um dado intervalo será maior, quanto maior a altura da curva nesse intervalo. No gráfico abaixo, o eixo *y* representa a densidade probabilística (que pode ser maior do que 1), não a probabilidade em si, e podemos pensar que os intervalos são infinitesimalmente pequenos.


```r
> den <- ggplot(casos, aes(notificados))
> den + geom_density(fill = 'white')
```

![plot of chunk 06-15](figures/06-15-1.png)

Ao mapearmos a variável `Species` na propriedade `fill`, as denisdades sobrepõem-se.


```r
> den <- ggplot(casos, aes(notificados))
> den + geom_density()
```

![plot of chunk 06-16](figures/06-16-1.png)

Para vissualizar melhor cada uma das densidades podemos controlar a transparência que varia entre 0 e 1.


```r
> den <- ggplot(casos, aes(notificados))
> den + geom_density(alpha = .5)
```

![plot of chunk 06-17](figures/06-17-1.png)


#### Outras geometrias comuns

* `geom_area`
* `geom_dotplot`
* `geom_freqploy`


### Gráficos de uma variável qualitativa

#### Barras

Os gráficos de barras representam cada uma das categorias de uma variável. A altura das barras representa a quantidade de observações.  


```r
> ggplot(casos, aes(sexo)) +
+   geom_bar(fill = 'lightblue')
```

![plot of chunk 06-18](figures/06-18-1.png)

A largura (ou separação) das barras é controlada com a propriedade `width`.


```r
> ggplot(casos, aes(sexo)) +
+   geom_bar(width = .5)
```

![plot of chunk 06-19](figures/06-19-1.png)

### Gráficos de duas variáveis quantitativas

#### Pontos

Os gráficos de pontos ou de dispersão representam com pontos o valor das duas variáveis para cada observação.  


```r
> ggplot(casos, aes(notificados, confirmados)) +
+   geom_point()
```

![plot of chunk 06-20](figures/06-20-1.png)

#### Linhas

Nos gráficos de linhas, para cada valor na amplitude de uma variável, existe um valor na outra variável. 


```r
> ggplot(casos, aes(semana, notificados)) +
+   geom_line()
```

![plot of chunk 06-21](figures/06-21-1.png)

No nosso banco, há 200 semanas e embora o registro de notificações tenha sido semanal, podemos ver que o gráfico anterior associa um número de notificações a qualquer ponto no período de 200 semanas (podemos associar um número de notificações à semana 3.4581). Como as notificações não foram registradas de forma contínua, o número de notificações no transcurso de uma semana, que não foi registrado, foi interpolado linearmente (estimado a partir de uma linha reta entre o número de notificações no começo da semana e o número de notificações no começo da semana seguinte). Isto fica mais claro ao combinarmos um gráfico de pontos e de linhas.


```r
> ggplot(casos, aes(semana, notificados)) +
+   geom_line() +
+   geom_point(color = 'red')
```

![plot of chunk 06-22](figures/06-22-1.png)

A interpolação linear não é a única forma de estimar o valor de uma variável. Podemos criar gráficos de passos que mantêm constante o valor da variável desde o último ponto de observação até o seguinte


```r
> ggplot(casos, aes(semana, notificados)) +
+   geom_step()
```

![plot of chunk 06-23](figures/06-23-1.png)


```r
> ggplot(casos, aes(semana, notificados)) +
+   geom_step() +
+   geom_point(color = 'red')
```

![plot of chunk 06-24](figures/06-24-1.png)

ou usar modelos estatísticos cuja explicação está além do escopo deste livro, mas a título de exemplo para quem tem familiaridade, são simples de incorporar especificando o método na função `geom_smooth`.


```r
> ggplot(casos, aes(semana, notificados)) +
+   geom_smooth(method = 'lm')
```

![plot of chunk 06-25](figures/06-25-1.png)

### Gráficos de uma variável qualitativa e uma quantitativa

#### Boxplot

Os boxplot representam a distribuição de uma variável quantitativa em cada uma das categorias da variável qualitativa. Um boxplot está composto por uma caixa que contém o 50% das observações; a base da caixa representa o quantil 25 (abaixo da base está o 25% das observações), uma linha dentro da caixa representa o quantil 50 ou mediana (abaxo da mediana está o 50% das observações) e o topo da caixa representa o quantil 75 (abaixo do topo está o 75% das observações). Portanto, a caixa contém o 50% dos dados (75% - 25%). Da caixa estendem-se "bigodes" que ajudam a identificar valores extremos (pontos além dos bigodes).


```r
> ggplot(casos, aes(sexo, notificados)) +
+   geom_boxplot()
```

![plot of chunk 06-26](figures/06-26-1.png)

O gráfico acima sugere a inexistência de valores extremos, pois não há pontos além dos bigodes. Para vermos como seria um boxplot com dados extremos, somemos 300 ao número de casos registrados nas 3 primeiras semanas, criemos os boxplot e depois voltemos aos valores originais.  


```r
> casos$notificados[1:3] <- casos$notificados[1:3] + 300
> ggplot(casos, aes(sexo, notificados)) +
+   geom_boxplot()
```

![plot of chunk 06-27](figures/06-27-1.png)

```r
> casos$notificados[1:3] <- casos$notificados[1:3] - 300
```

A soma de 300 aos 3 primeiros valores criou 2 valores extremos. Por que dois e não 3? Os valores extremos são os que se estendem além dos bigodes e a extensão dos bigodes é determinada por padrão da seguinte maneira:

O bigode superior vai do topo da caixa até o menor valor entre:  

* Quartil 75 + amplitude interquartil multiplicada * 1.5.
* Maior valor observado.  

O bigode inferior vai da base da caixa até o maior valor entre:  

* Quartil 25 - amplitude interquartil multiplicada * 1.5.
* Menor valor observado.  

Portanto, a soma de 300 fez com que 2 dos 3 primeiros valores fossem maiores do que *quartil 75 + amplitude interquartil multiplicada \* 1.5*.


```r
> quantile(casos$notificados, c(.75, .25))
```

```
   75%    25% 
203.00 101.75 
```

```r
> 203 + (203 - 101.75) * 1.5
```

```
[1] 354.875
```

```r
> casos$notificados[1:3] + 300
```

```
[1] 316 376 387
```

Para facilitar comparação das medianas de duas categorias diferentes, a propriedade `notch` cria entrâncias centradas nas medianas. A não superposição dessas entrâncias no eixo *y*, como no gráfico a seguir, sugere que as medianas são diferentes.


```r
> ggplot(casos, aes(sexo, notificados)) +
+   geom_boxplot(notch = T)
```

![plot of chunk 06-29](figures/06-29-1.png)

#### Barras

Os gráficos de barras também permitem visualizar a relação entre as categorias de uma variável e o valor de uma variável quantitativa, de tal forma que a altura das barras representa o valor da variável quantitativa. A função `geom_bar` nos permite isso mediante a propriedade `stat` que ao ser definida como `identity` define a altura das barras de cada categoria da variável mapeada em `x`, com base na variável mapeada em `y`.


```r
> ggplot(casos, aes(sexo, notificados)) +
+   geom_bar(stat = 'identity')
```

![plot of chunk 06-30](figures/06-30-1.png)

#### Outras gemotrias comuns

* `geom_violin`
* `geom_dotplot`

### Gráficos com três variáveis quantitativas

Criemos um banco fictício com coordenadas `x` e `y` variando de -10 a 10, e a cada coordenada atribuamos um valor para representar a concentração de um poluente, de tal forma que quanto mais central a coordenada, menor o valor.


```r
> poluente <- expand.grid(x = seq(-10, 10, l = 50), y = seq(-10, 10, l = 50))
> poluente$pol <- 0 + (poluente$x ^ 2 + poluente$y ^ 2)
```

#### Raster

Nos gráficos de raster ou de retículas, cada coordenada é o centro de uma célula ou pixel e o valor ou atributo de cada coordenada é mapeado em uma escala de cor.


```r
> ggplot(poluente, aes(x, y, fill = pol)) +
+   geom_raster()
```

![plot of chunk 06-32](figures/06-32-1.png)

#### Linhas de contorno

Nos gráficos com linhas de contorno ou isolinhas, cada linha representa um valor da variável mapeada (todas as coordenadas têm o mesmo valor se estão na mesma linha).


```r
> ggplot(poluente, aes(x, y)) +
+   geom_contour(aes(z = pol, color = ..level..))
```

![plot of chunk 06-33](figures/06-33-1.png)


```r
> ggplot(poluente, aes(x, y, fill = pol)) +
+   geom_raster() +
+   geom_contour(aes(z = pol, color = ..level..))
```

![plot of chunk 06-34](figures/06-34-1.png)

### Temas

Além das propriedades estéticas das geometrias, existem outros elementos modificáveis e não dependentes dos dados. O estilo ou *tema* dos gráficos é dado por alguns desses elementos e há temas predefinidos. Ao digitarmos `theme_` e dar um `TAB`, aparecerão os temas disponíveis, entre os quais estão os seguintes:


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme_bw()
```

![plot of chunk 06-35](figures/06-35-1.png)


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme_classic()
```

![plot of chunk 06-36](figures/06-36-1.png)


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme_dark()
```

![plot of chunk 06-37](figures/06-37-1.png)

Para maior controle dos elementos que compõem os eixos e as legendas, devemos usar a função `theme` em conjunto com *funções de elementos* que modificam elementos específicos.

### Eixos

#### Títulos

O título do eixos é dado pelo nome das variáveis mapeadas nas propriedades estéticas e é modificado pelas funções `xlab` e `ylab`.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   xlab('Alfa') +
+   ylab('Beta')
```

![plot of chunk 06-38](figures/06-38-1.png)

Caracteres especiais são interpretados pela função `expression`,


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   xlab(expression(alpha)) +
+   ylab(expression(beta))
```

![plot of chunk 06-39](figures/06-39-1.png)

enquanto sub e superescritos são incluídos pela função `bquote`.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   xlab(bquote(alpha[2])) +
+   ylab(bquote(beta^2))
```

![plot of chunk 06-40](figures/06-40-1.png)

O tamanho e a orientação são características do texto do elemento `axis.title`, modificadas pela função `element.text` para esse elemento. `axis.title` e `element.text` devem ser usados dentro da função `theme`.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(axis.title = element_text(size = 25, angle = 0))
```

![plot of chunk 06-41](figures/06-41-1.png)

É possível mudar só um dos eixos e outras características como a fonte (`face`), a justificação horizontal (`hjust`) e vertical (`vjust`), e a cor (`color`).


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(axis.title.x = element_text(size = 25, face = 'bold.italic',
+                                     hjust = .9))
```

![plot of chunk 06-42](figures/06-42-1.png)


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(axis.title.y = element_text(angle = 0, vjust = .1, color = 'red'))
```

![plot of chunk 06-43](figures/06-43-1.png)

#### Marcas

As características das marcas são definidas pelo elemento `axis.text` e modificadas pela função `element_text`.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(axis.text = element_text(size = 25, angle = 45))
```

![plot of chunk 06-44](figures/06-44-1.png)


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(axis.text.x = element_text(color = 'blue'))
```

![plot of chunk 06-45](figures/06-45-1.png)

#### Transposição

Ao acrescentarmos a função `coord_flip`, os eixos são transpostos.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   coord_flip()
```

![plot of chunk 06-46](figures/06-46-1.png)

#### Posição e etiqueta das marcas

Para alterar a posição e a etiqueta devemos usar a função `scale_x_continuous` com variáveis quantitativas e `scale_x_discrete` com variáveis qualitativas. Em qualquer caso, o argumento `breaks` define a posição e `levels` as etiquetas.


```r
> ggplot(casos, aes(sexo, notificados)) +
+   geom_boxplot(notch = T) +
+   scale_x_discrete(breaks = c('F', 'M'), labels = c('Fêmeas', 'Machos')) +
+   scale_y_continuous(breaks = seq(0, 300, by = 50))
```

![plot of chunk 06-47](figures/06-47-1.png)


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   scale_x_continuous(breaks = c(50, 150, 250)) +
+   scale_y_continuous(breaks = seq(0, 30, by = 5))
```

![plot of chunk 06-48](figures/06-48-1.png)

No caso de variáveis qualitativas, também podemos mudar as etiquetas mudando o nome das colunas do banco.

### Legendas

#### Título e marcas

Tanto o título como a posição e a etiqueta das marcas são definidos pelas funções da família `scale_` (por exemplo, `scale_fill_discrete`).


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   scale_color_discrete(name = 'Local', breaks = c('A', 'B'), labels = 1:2)
```

![plot of chunk 06-49](figures/06-49-1.png)

As características do texto do título e das marcas são modificadas seguindo a lógica  usada com os eixos.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(legend.title = element_text(size = 25),
+         legend.text = element_text(face = 'bold'))
```

![plot of chunk 06-50](figures/06-50-1.png)

#### Posição

Podemos posicionar a legenda acima (`top`), abaixo (`bottom`) ou aos lados (`left`, `right`).


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(legend.position = 'top')
```

![plot of chunk 06-51](figures/06-51-1.png)

Para posicioná-la dentro dentro dentro da área graficada, devemos especificar um par de coordenadas escaladas (cada eixo vai de 0 até 1).


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(legend.position = c(.1, .7))
```

![plot of chunk 06-52](figures/06-52-1.png)

Também podemos tirar a legenda.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point() +
+   theme(legend.position = 'none')
```

![plot of chunk 06-53](figures/06-53-1.png)

### Cores

A definição da cor é dada em termos de preenchimento (`fill`) e de contorno (`colour` ou `color`).  


```r
> ggplot(casos, aes(sexo, notificados)) +
+   geom_boxplot(fill = 'lightblue', color = 'red')
```

![plot of chunk 06-54](figures/06-54-1.png)

São várias as cores disponíveis e além do nome é possível usar os formatos RGB e hexadecimal. Quanto às cores disponíveis, há 657 opções cujos nomes são listados pela função `colors`.


```r
> str(colors())
```

```
 chr [1:657] "white" "aliceblue" "antiquewhite" ...
```

```r
> colors()[1:10]
```

```
 [1] "white"         "aliceblue"     "antiquewhite" 
 [4] "antiquewhite1" "antiquewhite2" "antiquewhite3"
 [7] "antiquewhite4" "aquamarine"    "aquamarine1"  
[10] "aquamarine2"  
```

```r
> ggplot(casos, aes(sexo, notificados)) +
+   geom_boxplot(fill = colors()[2], color = colors()[10])
```

![plot of chunk 06-55](figures/06-55-1.png)

As vezes é difícil escolher a cor usando a lista da função `colors`, mas se digitarmos *r color chart* na seção de imagens do google, aparecerão exemplos de cores com seus respectivos nomes, e formatos RGB e hexadecimal.  

Nos exemplos anteriores as cores foram definidas, mas nenhuma variável foi mapeada nas cores (as categorias fêmea e macho ficaram com a mesma cor). Se o objetivo for mapear uma variável em cores mudando o padrão, devemos usar as funções da família `scale_`. Com a função `scale_fill_manual` (`scale_color_manual`) podemos definir a cor para cada categoria.


```r
> ggplot(casos, aes(sexo, notificados, fill = local)) +
+   geom_boxplot()
```

![plot of chunk 06-56](figures/06-56-1.png)


```r
> ggplot(casos, aes(sexo, notificados, fill = local)) +
+   geom_boxplot() +
+   scale_fill_manual(values = c('lightblue', 'darkred'))
```

![plot of chunk 06-57](figures/06-57-1.png)


```r
> ggplot(casos, aes(sexo, notificados, color = local)) +
+   geom_boxplot() +
+   scale_color_manual(values = c('blue', 'green'))
```

![plot of chunk 06-58](figures/06-58-1.png)

No lugar de definirmos manualmente a cor de cada categoria, podemos escolher paletas de cores predefinidas (ver *r color brewer palette* no google), com a função `scale_fill_brewer` (`scale_color_brewer`).


```r
> ggplot(casos, aes(sexo, notificados, fill = local)) +
+   geom_boxplot() +
+   scale_fill_brewer(palette = 'Dark2')
```

![plot of chunk 06-59](figures/06-59-1.png)

Quando a variável mapeada é quantitativa, devemos usar a função `scale_fill_continuous` (`scale_color_continuous`) e definir os extremos da escala de cor.


```r
> ggplot(poluente, aes(x, y, fill = pol)) +
+   geom_raster() +
+   scale_fill_continuous(low = 'yellow', high = 'red')
```

![plot of chunk 06-60](figures/06-60-1.png)

Para incluir mais de duas cores na escala, podemos definir os extremos com mais de uma cor 


```r
> ggplot(poluente, aes(x, y, fill = pol)) +
+   geom_raster() +
+   scale_fill_continuous(low = c('yellow', 'green'), high = c('black', 'red'))
```

![plot of chunk 06-61](figures/06-61-1.png)

ou usar a função `scale_fill_gradientn` (`scale_color_gradientn`) com escalas de cores predefinidas pelas funções `rainbow`, `topo.colors` ou `terrain.colors`.


```r
> ggplot(poluente, aes(x, y, fill = pol)) +
+   geom_raster() +
+   scale_fill_gradientn(colors = rainbow(4))
```

![plot of chunk 06-62](figures/06-62-1.png)


```r
> ggplot(poluente, aes(x, y, fill = pol)) +
+   geom_raster() +
+   scale_fill_gradientn(colors = topo.colors(4))
```

![plot of chunk 06-63](figures/06-63-1.png)


```r
> ggplot(poluente, aes(x, y, fill = pol)) +
+   geom_raster() +
+   scale_fill_gradientn(colors = terrain.colors(4))
```

![plot of chunk 06-64](figures/06-64-1.png)

### Formas dos pontos e das linhas

Como no caso da cor e outras propriedades estéticas, temos como definir a forma dos pontos e das linhas ou mapear uma variável na forma.


```r
> ggplot(casos, aes(notificados, confirmados, color = local)) +
+   geom_point(shape = 3)
```

![plot of chunk 06-65](figures/06-65-1.png)


```r
> ggplot(casos, aes(notificados, confirmados,
+                   color = local, shape = local)) +
+   geom_point()
```

![plot of chunk 06-66](figures/06-66-1.png)


```r
> ggplot(casos, aes(semana, notificados)) +
+   geom_line(linetype = 2)
```

![plot of chunk 06-67](figures/06-67-1.png)


```r
> ggplot(casos, aes(semana, notificados, linetype = sexo)) +
+   geom_line()
```

![plot of chunk 06-68](figures/06-68-1.png)

Pontos e linhas disponíveis:


```r
> ggplot(data.frame(x = factor(1:25)), aes(x = x, y = x, shape = x)) +
+   geom_point(size = 5) +
+   scale_shape_manual(values = 1:25)
```

![plot of chunk 06-69](figures/06-69-1.png)


```r
> ggplot(data.frame(x = rep(0:6, 10), y = rep(0:6, e = 10)),
+        aes(x, y, linetype = factor(x))) +
+   geom_line(size = 2)
```

![plot of chunk 06-70](figures/06-70-1.png)

### Gráficos multivariados

No começo do capítulo criamos um gráfico multivariado.


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_grid(local ~ sexo)
```

![plot of chunk 06-71](figures/06-71-1.png)

A função `facet_grid` mapeia variáveis qualitativas nas linhas e colunas de uma matriz de gráficos. No exemplo anterior mapeamos o local nas linhas e o sexo nas colunas. Para mapear só nas linhas ou nas colunas, há que colocar um ponto na dimensão que não se quer mapear.


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_grid(. ~ sexo)
```

![plot of chunk 06-72](figures/06-72-1.png)


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_grid(sexo ~ .)
```

![plot of chunk 06-73](figures/06-73-1.png)

`facet_wrap` também gera gráficos multivariados e é conveniente quando se tem uma única variável condicionante e se quer controlar o número de colunas da matriz de gráficos


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_wrap(~ sexo, ncol = 1)
```

![plot of chunk 06-74](figures/06-74-1.png)

Quando se usam duas variáveis condicionantes, as combinações das categorias das mesmas são representadas em um único cabecalho (acima, no lugar de acima e à direita).


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_wrap(local ~ sexo, ncol = 3)
```

![plot of chunk 06-75](figures/06-75-1.png)

A forma mais simples de mudar as etiquetas de cada painel é definindo as etiquetas do fator usado para condicionar o gráfico.


```r
> levels(casos$sexo)
```

```
[1] "F" "M"
```

```r
> casos$sexo <- factor(as.character(casos$sexo), levels = c('F', 'M'),
+                      labels = c('Fêmeas', 'Machos'))
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_grid(sexo ~ .)
```

![plot of chunk 06-76](figures/06-76-1.png)

Mudando a ordem dos níveis do fator também mudamos a ordem dos paineis.


```r
> levels(casos$sexo)
```

```
[1] "Fêmeas" "Machos"
```

```r
> casos$sexo <- factor(casos$sexo, levels = c('Machos', 'Fêmeas'))
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_grid(sexo ~ .)
```

![plot of chunk 06-77](figures/06-77-1.png)

Com o elemento `strip.text.x` modificamos a apariência das etiquetas dos paineis.


```r
> ggplot(data = casos, aes(notificados, confirmados)) +
+   geom_point() +
+   facet_grid(. ~ sexo) + 
+   theme(strip.text.x = element_text(size = 8, colour = 'orange', angle = 45))
```

![plot of chunk 06-78](figures/06-78-1.png)

### Sintaxe abreviada

O `ggplot2` tem a função `qplot` que usa uma sintaxe abreviada. Os dois primeiros argumentos são `x` e `y`, e a geometria é escolhida automaticamente.


```r
> qplot(casos$notificados, casos$confirmados)
```

![plot of chunk 06-79](figures/06-79-1.png)

Se a geometria escolhida não é adequada, o argumento `geom` permite definir a gemoetria desejada.


```r
> qplot(casos$sexo, casos$confirmados)
```

![plot of chunk 06-80](figures/06-80-1.png)


```r
> qplot(casos$sexo, casos$confirmados, geom = 'boxplot')
```

![plot of chunk 06-81](figures/06-81-1.png)

Outras propriedades estéticas também são definíveis dentro de `qplot`,


```r
> qplot(casos$notificados, casos$confirmados, color = casos$sexo)
```

![plot of chunk 06-82](figures/06-82-1.png)

e alterantivamente, podemos especificar só o nome das variáveis se especificamos o banco no argumento `data`.


```r
> qplot(notificados, confirmados, data = casos)
```

![plot of chunk 06-83](figures/06-83-1.png)

