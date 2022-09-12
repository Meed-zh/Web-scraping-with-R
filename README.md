# Web-scraping-with-R
In this repository, I will extract data from a job website in France using R language with Rvest package

## Library

```{r, eval = FALSE}
library(tidyverse)
library(httr)
library(rvest)
```

## Table creation

```{r, eval = FALSE}
tableau_JOB <- tibble(
  TITRE = character(),
  ENTREPRISE = character(),
  LIEUX = character(),
  TEMPS = character()
)
```

## Lister les URLs

```{r, eval = FALSE}
list_url <- paste0("https://www.optioncarriere.com/emploi-data-analyst.html?p=",c(1:71))
```

## L'extraction des données par la création d'une boucle 

```{r, message = FALSE}
for (url in list_url){
  
  print(url)
  
  # Extraction de la page web
  page_offre_html <- try(content(GET(url)),silent=TRUE)
  
  if(class(page_offre_html)[1] == "try-error") next
  
  # LISTER LES OFFRES D'EMPLOIS AVEC LA BALISE div
  liste_offre <- html_elements(page_offre_html,xpath = "//article[@class='job clicky']")
  
  
  # BOUCLE SUR LES OFFRES D'EMPLOIS
  for (k in 1:length(liste_offre)){
    
    # Donner l'id de l'extraction en cours
    print(k)
    
    # SELECTION DE L'OFFRE [k]
    offre_emplois <- liste_offre[[k]]
    
    
    # EXTRACTION DES INFORMATIONS : 	#TITRE DE L'OFFRE, 
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
    tableau_JOB <- tableau_JOB %>% 
      add_row(TITRE = titre,
              ENTREPRISE = entreprise,
              LIEUX = lieux,
              TEMPS = temps)
    
    Sys.sleep(1)
  }
}
```
