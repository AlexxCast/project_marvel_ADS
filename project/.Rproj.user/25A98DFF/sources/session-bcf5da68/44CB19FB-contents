install.packages(c("tidyverse", "igraph", "tidytext", "widyr", "tidygraph", "ggraph", "ggrepel", "RColorBrewer"))
library(tidyverse)
library(igraph)
library(tidytext)
library(widyr)

library(tidygraph)
library(ggraph)
library(ggrepel)
library(RColorBrewer)
exists("graph_from_data_frame")
colnames(read.csv("superHeroes.csv"))
db_leg <- read.csv("hero-network.csv")
db_eroi <- read.csv("superHeroes.csv")
glimpse(db_leg)
glimpse(db_eroi)

marvel_grafo <- graph_from_data_frame(db_leg, directed = FALSE)
E(marvel_grafo)$weight <- 1
marvel_grafo <- igraph::simplify(marvel_grafo, edge.attr.comb = list(weight = "sum"))

# weights = NA: conservo i pesi come informazione ma calcolo NON pesato,
# per coerenza con la community detection
punteggio_betweenness <- betweenness(marvel_grafo, normalized = TRUE, weights = NA)

db_nodi <- data.frame(
  Character = V(marvel_grafo)$name,
  Betweenness = punteggio_betweenness
)

communities <- cluster_louvain(marvel_grafo, weights = NA)
db_nodi$Community <- membership(communities)

# per uniformare i nomi all'interno delle colonne dei database ed evitare inconsistenze
db_nodi <- db_nodi %>%
  mutate(join_key = str_replace_all(Character, "/.*", "")) %>%   # taglia da "/" in poi
  mutate(join_key = str_to_lower(join_key)) %>%
  mutate(join_key = str_replace_all(join_key, "[[:punct:]]", "")) %>%
  mutate(join_key = str_squish(join_key))

db_eroi <- db_eroi %>%
  filter(Publisher == "Marvel Comics") %>%          # tieni solo Marvel: pulisce omonimie
  mutate(join_key = str_to_lower(Name)) %>%
  mutate(join_key = str_replace_all(join_key, "[[:punct:]]", "")) %>%
  mutate(join_key = str_squish(join_key))%>%
  distinct(join_key, .keep_all = TRUE)     # una sola scheda per nome

db_analisi <- inner_join(db_nodi, db_eroi, by = "join_key") %>%
  mutate(
    Intelligence = as.numeric(Intelligence),
    Strength     = as.numeric(Strength)
  ) %>%
  select(Character, Betweenness, Community, Intelligence, Strength, Group.Affiliation,Publisher) %>%
  
  drop_na(Intelligence, Strength)

glimpse(db_analisi)

db_contollo <- anti_join(db_nodi, db_eroi, by = "join_key") %>% 
  pull(Character) %>% 
  head(30)
print(db_contollo)

#cerco la corelazione tra inteligenza e forza con la betinnwes
cor(db_analisi$Betweenness, db_analisi$Intelligence, method = "pearson")
cor(db_analisi$Betweenness, db_analisi$Strength,     method = "pearson")

cor.test(db_analisi$Betweenness, db_analisi$Intelligence, method = "pearson")
cor.test(db_analisi$Betweenness, db_analisi$Strength,     method = "pearson")

modello <- lm(Betweenness ~ Intelligence + Strength, data = db_analisi)
summary(modello)

db_analisi %>% arrange(desc(Betweenness)) %>% head(10)

ggplot(db_analisi, aes(x = Intelligence, y = Betweenness)) +
  geom_point()+
  geom_smooth(method = "lm")+
  labs(title = "studio intelligenza", x = "intelligenza", y = "cardinaliuta")        

ggplot(db_analisi, aes(x = Strength, y = Betweenness)) +
  geom_point()+
  geom_smooth(method = "lm")+
  labs(title = "studio forza", x = "forza", y = "cardinaliuta")      

db_analisi %>%
  group_by(Community) %>%        # "considera i personaggi gruppo per gruppo"
  summarise(                      # "per ogni gruppo, calcola una riga di sintesi"
    Numero_Personaggi= n(),
    inteligenza_Media= mean(Intelligence),
    Forza_Media= mean(Strength),
    Betweenness_Media= mean(Betweenness)
  )%>% 
  filter( Numero_Personaggi  >= 5)  


token_affiliazioni <- db_analisi %>%
  select(Community, Group.Affiliation) %>%
  separate_rows(Group.Affiliation, sep = ",") %>%    # spezza sulla virgola
  mutate(termine = str_squish(str_to_lower(Group.Affiliation)))%>%   # pulisci
  filter(termine != "") # togli i risultati vuoti 

# AI inizio

tfidf_affiliazioni <- token_affiliazioni %>%
  count(Community, termine) %>%              # conta: quante volte ogni termine per comunità
  bind_tf_idf(termine, Community, n)         # applica TF-IDF
glimpse(tfidf_affiliazioni)

nomi_comunita <- tfidf_affiliazioni %>%
  filter(n >= 2) %>%              # scarta i termini che compaiono una volta sola (rumore)
  group_by(Community) %>%
  slice_max(tf_idf, n = 1)      # per ogni comunità, la riga col tf_idf più alto
# AI Fine
# ri-tokenizzo tenendo il PERSONAGGIO come contesto
token_per_personaggio <- db_analisi %>%
  select(Character, Group.Affiliation) %>%
  separate_rows(Group.Affiliation, sep = ",") %>%
  mutate(termine = str_squish(str_to_lower(Group.Affiliation))) %>%
  filter(termine != "", termine != "-")

# conto le coppie di affiliazioni condivise dagli stessi personaggi
# AI inizio
coppie_affiliazioni <- token_per_personaggio %>%
  pairwise_count(termine, Character, sort = TRUE)
# AI Fine
glimpse(coppie_affiliazioni)

nomi_comunita <- tfidf_affiliazioni %>%
  filter(n >= 2) %>%
  group_by(Community) %>%
  slice_max(tf_idf, n = 1)

top15 <- db_analisi %>%
  arrange(desc(Betweenness)) %>%
  head(15) %>%
  pull(Character)

# AI: ritaglio dalla rete completa il sotto-grafo dei soli top 15 boundary spanner
grafo_ponti <- induced_subgraph(marvel_grafo, vids = V(marvel_grafo)$name %in% top15)

# AI: converto in tidygraph e aggancio a ogni nodo le sue statistiche da db_analisi
grafo_ponti <- grafo_ponti %>%
  as_tbl_graph() %>%
  activate(nodes) %>%
  left_join(db_analisi, by = c("name" = "Character")) %>%   # <-- aggiungi %>% qui
  left_join(nomi_comunita, by = "Community")                # ora è dentro la catena

ggraph(grafo_ponti, layout = "circle") +
  geom_edge_link(aes(alpha = weight), color = "gray50") +
  geom_node_point(aes(size = Betweenness, color = termine)) +
  geom_node_text(aes(label = name), repel = TRUE) +
  labs(title = "I boundary spanner Marvel", color = "Comunità", size = "Betweenness")   # etichette, colori, ecc.

