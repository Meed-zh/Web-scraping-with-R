# Web-scraping-with-R
In this repository, I will extract data from a job website in France using R language with Rvest package

## Library
##### Installation des packages

```{r, eval = FALSE}
install.packages('tidyverse')
install.packages('httr')
install.packages('rvest')
```
#### Chargement des packages

```{r, eval = FALSE}
library(tidyverse)
library(httr)
library(rvest)
```

#### Je commence d'abord par la création du tableau « DF_JOB » dans lequel je vais collecter les données. Ensuite, avec la fonction « tibble » du package tidyverse, je déclare les colonnes et les types des variables « character » 

```{r, eval = FALSE}
DF_JOB <- tibble(
  OFFRE = character(),
  DESCRIPTION = character(),
  ENTREPRISE = character(),
  LIEUX = character(),
)
```

#### La création de l'objet « list_url » dans lequel je stocke la liste des urls à extraire 

```{r, eval = FALSE}
list_url <- paste0("https://www.optioncarriere.com/emploi-data.html?p=",c(1:100))
```

## L'extraction des données en utilisant une boucle « for »

```{r, message = FALSE}

# D'abord, je crée une boucle sur les urls listés dans « list_url » pour extraire le contenu de chaque url en html.  
for (url in list_url){
  
  print(url)
  
  # Extraction de la page web
  page_offre_html <- try(content(GET(url)),silent=TRUE)
  
  if(class(page_offre_html)[1] == "try-error") next
  
  # Une fois le contenu des urls est recuperé, je vais lister dans « list_offre » les offres d'emploi qui se trouve dans la balise "//article[@class='job clicky']"
  liste_offre <- html_elements(page_offre_html,xpath = "//article[@class='job clicky']")
  
  
  # BOUCLE SUR LES OFFRES D'EMPLOIS
  for (k in 1:length(liste_offre)){
    
    # Donner l'id de l'extraction en cours
    print(k)
    
    # SELECTION DE L'OFFRE [k]
    offre_emplois <- liste_offre[[k]]
    
    
    # EXTRACTION DES INFORMATIONS : 	
        #TITRE DE L'OFFRE, 
        #ENTREPRISE, 
        #VILLE, 
        #TEMPS DE PUBLICATION DE L'OFFRE
    
    titre <- html_element(offre_emplois, xpath = ".//h2") %>%
      html_text %>%
      str_trim 
    
    entreprise <- html_element(offre_emplois, xpath = ".//p[@class='company']") %>% 
      html_text %>%
      str_trim %>%
      str_remove_all("\n") %>%
      str_to_lower
    
    
    lieux <- html_element(offre_emplois, xpath = ".//ul[@class='location']") %>%
      html_text %>%
      str_trim %>%
      str_remove_all("\n") %>%
      str_to_lower
    
    temps <- html_element(offre_emplois, xpath = ".//span[@class='badge badge-r badge-s badge-icon']") %>% 
      html_text %>%
      str_trim %>%
      str_remove_all("\n") %>%
      str_remove_all("Il y a ") %>%
      str_to_lower
    
    
    # INSERTION DES DONNEES SCRAPPER SUR LE TABLEAU DECLARE EN AMONT
    DF_JOB <- DF_JOB %>% 
      add_row(OFFRE = titre,
              DESCRIPTION = description,
              ENTREPRISE = entreprise,
              LIEUX = lieux)
    
    Sys.sleep(1)
  }
}
```
