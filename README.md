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

#### D'abord, la création du tableau « DF_JOB » dans lequel je collecte les données. Ensuite, avec la fonction « tibble » du package tidyverse, je déclare les colonnes et les types des variables « character » 

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

# D'abord, je crée une boucle sur les urls listés dans « list_url » pour extraire le contenu de chaque 
# url en html.  
for (url in list_url){
  
  print(url)
  
  # EXTRACTION LA PAGE WEB 
  page_offre_html <- try(content(GET(url)),silent=TRUE)
  
  if(class(page_offre_html)[1] == "try-error") next
  
  # Une fois le contenu des urls est récupéré, je vais lister dans « list_offre » les offres d'emploi qui 
  # se trouvent dans la balise « //article[@class='job clicky'] »
  liste_offre <- html_elements(page_offre_html,xpath = "//article[@class='job clicky']")
  
  
  # Création d'une deuxième boucle sur la liste des offres d'emplois « list_offre »
  for (k in 1:length(liste_offre)){
    
    # Donner l'id de l'extraction en cours
    print(k)
    
    # SELECTION DE L'OFFRE [k]
    offre_emplois <- liste_offre[[k]]
    
    # Excration des informations : 	
    # « Titre de l'offre », « Description de l'offre », « Entreprise », « Lieux »
    # D'abord, je recupere le contenu en html avec la fonction « html_element » ensuite je garde uniquement   
    # le contenu text dans la balise html avec la fonction « html_text »
    
    # « Titre de l'offre » :
    titre <- html_element(offre_emplois, xpath = ".//h2") %>%
      html_text %>%
      str_trim %>% # supprimer l'espace 
      str_remove_all("\n") %>% # supprimer les caractères spéciaux 
      str_to_lower # Convertir tous les caractères alphabétiques en minuscules
    
    # « Entreprise » :
    entreprise <- html_element(offre_emplois, xpath = ".//p[@class='company']") %>% 
      html_text %>%
      str_trim %>%
      str_remove_all("\n") %>%
      str_to_lower
    
    # « Lieux » :
    lieux <- html_element(offre_emplois, xpath = ".//ul[@class='location']") %>%
      html_text %>%
      str_trim %>%
      str_remove_all("\n") %>%
      str_to_lower
    
    # « Description de l'offre » :
    # Extraction de l'url de chaque offre d'emploi pour récupérer la description de l'offre
    url_description <- html_attr(html_element(offre_emplois, xpath = ".//h2/a"),name="href")
    url_description <- paste("https://www.optioncarriere.com",url_description,sep = "")
    
    page_description <- content(GET(url_description))
    
    if(class(page_description)[1] == "try-error") next
    
    description <- html_text(html_element(page_description, xpath = ".//section[@class='content']")) %>%
      str_trim %>%
      str_remove_all("[.:' ;,?=&^!/-•]<>+\n") %>%
      str_to_lower
   
    # Insértion des données scrappées dans le tableau
    DF_JOB <- DF_JOB %>% 
      add_row(OFFRE = titre,
              DESCRIPTION = description,
              ENTREPRISE = entreprise,
              LIEUX = lieux)
    
    # Pour éviter tout blocage, la fonction « Sys.sleep(1) » permet de faire une pause d'une seconde
    # après la fin d'extraction de données de chaque url 
    Sys.sleep(1)
  }
}
```
