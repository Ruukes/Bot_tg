

# Loading libraries

```{r}
library(telegram.bot)
library(stringr)
library(hash)


```



# Bot token and updating

```{r}

bot_token <- ""

bot <- Bot(token=bot_token)
updater= Updater(bot_token)
dispatcher <- updater$dispatcher
```



# Functions for bot

```{r}
spheres <- c(occupations_final$major %>% unique())
spheres <- c(spheres, "Не интересует конкретная сфера")
exp <- c("Нет опыта или меньше 1 года", "От 1 до 3 лет", "От 3 до 6 лет",
         "Больше 6 лет")

start_keyboard <- ReplyKeyboardMarkup(
  keyboard = list(
    list(KeyboardButton("Общение с людьми")),
    list(KeyboardButton('Программирование')),
    list(KeyboardButton('Техническое')),
    list(KeyboardButton('Предоставление услуг')),
    list(KeyboardButton('Маркетинг')),
    list(KeyboardButton('Не интересует конкретная сфера'))
  ),
  one_time_keyboard = TRUE
)

exp_keyboard <- ReplyKeyboardMarkup(
  keyboard = list(
    list(KeyboardButton("Нет опыта или меньше 1 года")),
    list(KeyboardButton('От 1 до 3 лет')),
    list(KeyboardButton('От 3 до 6 лет')),
    list(KeyboardButton('Больше 6 лет'))
  ),
  one_time_keyboard = TRUE
)

j = hash()
j[["occ"]] <- "Программирование"
j[["exp"]] <- "Нет опыта или меньше 1 года"
j[["skills"]] <- c("шитье", "вязание")
```




```{r}

start_h <- function(bot, update) {
  bot$sendMessage(update$message$chat_id, 
                  text = "Привет! Я бот для поиска работы, выберите сферу своей деятельности.",
                  reply_markup = start_keyboard)
  

}


start_handler <- function(bot, update){
  
  tex <- trimws(update$message$text)
  
  if (tex %in% spheres) {
    
    j[["occ"]] <- tex
    

    bot$sendMessage(chat_id = update$message$chat_id,
                    text = "Отлично! Теперь выберите как долго вы работаете по своей специальности. ",
                    reply_markup = exp_keyboard)
  }
  
  
  if (tex %in% exp) {
    
    j[["exp"]] <- tex
    

    bot$sendMessage(chat_id = update$message$chat_id,
                    text = "Отлично! Теперь напишите все ваши навыки через пробел.")
  }
  
  if ((tex %in% spheres == F) & (tex %in% exp == F) & (tex != "/start") & ((tex != "/more")) ) {
    
    skills <- strsplit(trimws(update$message$text), split=' ', fixed=TRUE)
    
    j[["skills"]] <- skills
    
    print(update$message$chat_id)
    
    t <- get_occ(j[["skills"]][[1]], j[["exp"]], j[["occ"]])
    
    print(j)
    print(t)
    
    j[["rep"]] <- 0
    
    if (length(t$major) == 0) {
      bot$sendMessage(chat_id = update$message$chat_id,
                      "Нет подходящих вакансий по этим данным. Попробуйте ввести другие данные /start.")
    } else {
      as <- c()
      print(length(t[1:3] %>% na.omit()))
            
      
      if (length(t$major) < 3) {
        f <- length(t$major)
      } else {
        f <- 3
      }
      
      for (i in 1:f) {
        g <- str_c(t$name[i], t$companyname[i], sep = "  ")
        as <- c(as, g)
        }
      for (i in 1:f) {
        bot$sendMessage(chat_id = update$message$chat_id,
                        as[i])
        }
      bot$sendMessage(chat_id = update$message$chat_id,
                "Можете проверить актуальность на https://hh.ru")
      bot$sendMessage(chat_id = update$message$chat_id,
      "Чтобы начать заново подбор вакансий введите/start.\nЧтобы посмотреть другие вакансии по этим же данным введите /more")
      
      
      }
  }
  
}

more_h <- function(bot, update) {
  t <- get_occ(j[["skills"]], j[["exp"]], j[["occ"]])
  as <- c()
  
  j[["rep"]] <- j[["rep"]] + 1
  
  r <- j[["rep"]]
  print(r)
  print(length(t$major))
  
  if (r * 3 +1 > length(t$major)){
    bot$sendMessage(chat_id = update$message$chat_id,
                    "Большей вакансий нет")
    } else if ((r * 3 + 3 >= length(t$major)) & (r * 3 + 1 <=              
                                                 length(t$major))) {
      for (i in (r * 3 +1):(length(t$major))) {
        g <- str_c(t$name[i], t$companyname[i], sep = "  ")
        as <- c(as, g)
      }
      
      for (i in 1:3) {
        bot$sendMessage(chat_id = update$message$chat_id,
                        as[i])
      }
      
      bot$sendMessage(chat_id = update$message$chat_id,
                  "Можете проверить актуальность на https://hh.ru")
      
      } else {
        for (i in (r * 3 +1):(r * 3 + 3)) {
          g <- str_c(t$name[i], t$companyname[i], sep = "  ")
          as <- c(as, g)
        }
        
        for (i in 1:3) {
          bot$sendMessage(chat_id = update$message$chat_id,
                          as[i])
        }
        
        bot$sendMessage(chat_id = update$message$chat_id,
                  "Можете проверить актуальность на https://hh.ru")
      }
  
  
  
  
}


```


# Function for recommendation system

```{r}
get_occ = function(x,y,m,b=3){
  
  if (y == "Нет опыта или меньше 1 года") {
    y = "noExperience"
  } else if (y == "От 1 до 3 лет") {
    y = "between1And3"
  } else if (y == "От 3 до 6 лет") {
    y = "between3And6"
  } else if (y == "Больше 6 лет") {
    y = "moreThan6"
  }
  
  target <- c(x)%>%str_to_lower()
  occup_major = select(occupations,id,major)
  filtered_clust<- inner_join(clust_final,occup_major)
  
  if (m != "Не интересует конкретная сфера") {
    filtered_clust = filtered_clust %>%
      dplyr::filter(experience.id == y & major == m) %>%
      unique() 
    } else {
        filtered_clust = filtered_clust %>%
          dplyr::filter(experience.id == y) %>%
          unique()
      }
  
  filtered_clust$skills = str_to_lower(filtered_clust$skills)
  
  filtered_clust2 <-dplyr::filter(filtered_clust,str_detect(filtered_clust$skills_sep_vac, target)) 
  
  
  if (nrow(filtered_clust2)==0) {
    recommend = c()} else {
  filtered_clust=filtered_clust%>%select(id, skills_sep_vac, skillsvv)
  ent_val=data.frame(40999992,target)
  names(ent_val) <- c("id", "skills")
  ent_val = ent_val %>%
    mutate(skills_sep_vac = strsplit(gsub("[][\"]", "", skills), ",")) %>%
    unnest(skills_sep_vac) %>%
    mutate(skillsvv = 1)%>%
    select(-skills) 
  comb_vac <- rbind(filtered_clust, ent_val)
  comb_vac= comb_vac %>%
    pivot_wider(names_from = skills_sep_vac, 
                                     values_from = skillsvv, 
                                     values_fill = 0) 
  rownames= comb_vac$id
  comb_vac = comb_vac%>%
    select(-id)
  rownames(comb_vac) = rownames
  sim = lsa::cosine(t(as.matrix(comb_vac)))
  diag(sim) = 0
  simCut = sim[,as.character(40999992)]
  mostSimilar = head(sort(simCut, decreasing = T), n = b)
  a = which(simCut %in% mostSimilar, arr.ind = TRUE, useNames = T)
  index = arrayInd(a, .dim = dim(sim))
  index[,1]
  result = rownames(sim)[index[,1]]
  mostSimilar = data.frame(id = as.numeric(result),
                           similar = simCut[index])
  recommend=mostSimilar %>%
    left_join(occupations_final) %>%
    select(name, id, similar,major,key_skills, employment.id) %>%
    top_n(similar,n=b)%>%
    arrange(-similar)
  }
  recommend
  }



```
