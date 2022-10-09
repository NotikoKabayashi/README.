# README.
main



from flask import Flask
from json import load
import gensim
import os
import nltk
from nltk.corpus import stopwords
import pymorphy2
from gensim.models.word2vec import Word2Vec

# pip install gensim==3.7.0 nltk pymorphy2
for st in range(0, 1):
    page = requests.get("https://www.buhonline.ru/pub/news")
    soup = BeautifulSoup(page.text, 'lxml')
    data = soup.find_all('div', class_='row-table row-table-100 m-l-0 m-r-0')
    news_ria = [0] * len(data)
    news_ria_link = [0] * len(data)
    print(len(data))
    i = 0

    for news in data:
        link_new = 'https://www.buhonline.ru' + news.find('a').get('href')


        for j in range(1, 2):
            new_str = news.find('a').getText('')
            new_str = new_str.replace('\xa0', " ")


        print(link_new)
        print(new_str)
        news_ria[i] = new_str
        news_ria_link[i] = link_new
        i = i + 1
    print(i)
    print(news_ria)
    print(news_ria_link)

for st in range(0, 1):
    page = requests.get("https://1prime.ru/business/")
    soup = BeautifulSoup(page.text, 'lxml')
    data = soup.find_all('article', class_='rubric-list__article rubric-list__article_default')
    news_rbc = [0] * len(data)
    news_rbc_link = [0] * len(data)
    print(len(data))
    i = 0

    for news in data:
        link_new = 'https://1prime.ru/' + news.find('a').get('href')


        for j in range(1, 2):
            new_str = news.find('h2').getText('')
            new_str = new_str.replace('\n',"")


        print(link_new)
        print(new_str)
        news_rbc[i] = new_str
        news_rbc_link[i] = link_new
        news_rbc[i] = new_str
        i = i + 1
    print(i)

    print(news_rbc)
    print(news_rbc_link)

for st in range(0, 1):
    page = requests.get("https://www.it-world.ru/")
    soup = BeautifulSoup(page.text, 'lxml')
    data = soup.find_all('div', class_='news-time')
    news_habr = [0] * len(data)
    news_habr_link = [0] * len(data)
    print(len(data))
    i = 0

    for news in data:
        link_new = 'https://www.it-world.ru/' + news.find('a').get('href')


        for j in range(1, 2):
            new_str = news.find('div', class_='news__text').getText('')
            new_str = new_str.replace('\xa0', "")


        print(link_new)
        print(new_str)
        news_habr[i] = new_str
        news_habr_link[i] = link_new
        i = i + 1
    print(i)
    print(news_habr)
    print(news_habr_link)

full_news = news_rbc + news_ria + news_habr
print(full_news)
full_news_links = news_rbc_link + news_ria_link + news_habr_link
print(full_news_links)

morph = pymorphy2.MorphAnalyzer()
nltk.download('stopwords')
stopWords = set(stopwords.words('russian') + ['если', 'быть', 'который', 'для', 'хотеть', 'при', 'мочь', 'новый','это','из-за', 'весь', 'свой','всё', 'нужно', 'получить', 'работа'])

class Role:
    def __init__(self, name, keys, model = '', stopwords = [], news = {}):
        self.name = name
        self.model = Word2Vec.load(model)
        self.keys = get_keys(keys, self.model)
        self.stopwords = stopwords
        self.news = news

def filtrated(new):
    Filtered = []
    for w in new.split():
        if morph.parse(w)[0].normal_form not in stopWords:
          Filtered.append(morph.parse(w)[0].normal_form) #лемматизируем и удаляем стоп слова
    return ''.join(f"{str(x)} " for x in Filtered)


def learn(name='Название модели', file='Файл, на основе которого создаем модель'):
    with open(file, encoding = 'utf-8', mode ='r') as f:
        news = f.read().splitlines()

    text = ''
    for n in news:
        text = f"{text}\n{filtrated(n)}"

    f = 'news_filtr.txt'
    with open('news_filtr.txt', encoding = 'utf-8', mode ='w') as fl:
            fl.writelines(text)
    data = gensim.models.word2vec.LineSentence(f)
    model = gensim.models.Word2Vec(data, size=300, window=2, min_count=3, sg=0)
    model.init_sims(replace=True)
    model.save(f"{name}.model")
    os.path.remove(f)
    return model

def get_keys(data=['Сюда вводятся ключевые слова, которые мы ассоциируем с ролью'],model='Используемая модель'):
    words = [morph.parse(word)[0].normal_form for word in data] #леммитизируем словарь
    summ = {}
    for word in words:
        # есть ли слово в модели? Может быть, и нет
        if word in model:
            summ[word] = 1
            # выдаем 10 ближайших соседей слова:
            for i in model.most_similar(positive=[word], topn=3):
                # слово + коэффициент косинусной близости
                summ[i[0]] =  i[1]
    return summ

# model = learn('gendir','data_set.txt') #создание модели и обучение ее из файла
Buhgalter = Role(
    name='Бухгалтер',
    keys=['налог', 'льгота', 'отсрочка', 'доход', 'учет', 'декларация', 'отчетность', 'справка', 'форма', 'бухгалтер'],
    model='mix.model',
    stopwords=['США', 'Доллар', 'Евро', 'Зарубеж', 'Иностранный'],
    news={})

Gendir = Role(
    name='Гендиректор',
    keys=['финансы', 'недвижимость', 'рост', 'ставка', 'процент', 'ЦБ' ,'центробанк', 'важно' , 'банк', 'предприниматель'],
    model='mix.model',
    news={})

It = Role(
    name='Специалист IT-отдела',
    keys=['IT', 'NFT', 'Программа', 'Информация', 'Технологии', 'Интеллект', 'ИИ', 'Data', 'Айтишник','Цифровой','программирования'],
    model='mix.model',
    news={})
roles = [Buhgalter, Gendir, It]

# берем новости из файла и загружаем построчно в load_news
for k  in range(1,2):
    load_news = full_news
    print(load_news)


for new in load_news:
    news_summ = {}
    for role in roles: #перебираем роли
        summ = 0
        # лемматизириуем и удаляем лишнее из новости
        newf = filtrated(new)
        keys_new = get_keys(newf.split(), role.model)
        for k in keys_new.keys():
            if k in role.keys.keys():
                summ = summ + keys_new[k]
        news_summ[role] = summ
    maximum = max(news_summ, key=lambda key: news_summ[key])
    for role in roles:
        if role is maximum:
            role.news[new] = news_summ[maximum]

# вывод 3 лучших новостей
for role in roles:
    items = dict(sorted(role.news.items(), key=lambda item: item[1], reverse=True)[:3])
    print(f"Новости для роли '{role.name}': ")
    for i in items.keys():
        print(i) #Здесь хранится новость в чистом виде
    print()
@app.route("/Gendir")
def gendir():
    # вывод 3 лучших новостей
    items = dict(sorted(roles[1].news.items(), key=lambda item: item[1], reverse=True)[:3])
    news = []
    for new in items.keys():
        news.append(new)
    return f'''<!DOCTYPE html>
<!-- saved from url=(0052)file:///C:/Users/op sky/Desktop/Verstka/index.html -->
<html lang="en"><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

        <link rel="stylesheet" href="C:/Users/op sky/Desktop/Verstka/main.css">
        <link href="file:///C:/Users/op sky/Desktop/Verstka/Images/Roboto-BlackItalic.ttf" rel="stylesheet">
        <title>VTB </title>
    </head> 
    <body>
        <header class="header">
            <div class="container">
            </div>
        <div class="header__logo">
            <img src="C:/Users/op sky/Desktop/Verstka/2.jpg" alt="">
        </div>

        </header>



<div class="about_img">
    <img src="C:/Users/op sky/Desktop/Verstka/1.jpg" alt="">       <!--ГЛАВНАЯ КАРТИНКА-->

</div>
          <div class="about_img">
    <img src="C:/Users/op sky/Desktop/Verstka/5.jpg" alt="">                                                  
</div>

<div class="news">
<div class="news_item">
    <div class="about_img">
        <img src="C:/Users/op sky/Desktop/Verstka/3.jpg" alt="">
    </div>
     <div class="about__text">Подробнее</div>    
</div>
<div class="news">
    <div class="news_item">
        <div class="about_img">
            <img src="C:/Users/op sky/Desktop/Verstka/3.jpg" alt="">
        </div>
        <div class="about__text">Подробнее</div> 
    </div>

    <div class="news">
        <div class="news_item">
            <div class="about_img">
                <img src="C:/Users/op sky/Desktop/Verstka/3.jpg" alt="">
            </div>
            <div class="about__text">Подробнее</div> 
        </div>

</div>
</div></div></body></html>'''


print("\x1b]8;;http://127.0.0.1:5000/Buhgalter\aCtrl+Click here\x1b]8;;\a")
if (__name__ == '__main__'):
    app.run(debug=True)
app.config['SEND_FILE_MAX_AGE_DEFAULT'] = 0












IMAGES.HTML












<!DOCTYPE html>
<html lang="ru">
    <head>
        <meta charset="UTF-8">
       <link rel="stylesheet" type="text/css" href="{{ url_for( 'static', filename='Style.css', v=1)}}">
        <link href="Images/Roboto-BlackItalic.ttf" rel="stylesheet">
        <title>VTB </title>
    </head>
    <body>
        <header class="header">
            <div class="container">
            </div class="header_inner">
        <div class="header__logo">
            <img src="C:/Users/op sky/PycharmProjects/server2/ВТБ.jpg" alt="">
        </div>
        <nav class="nav">
            <a class="nav__link" herf="#">Частным лицам</a>
            <a class="nav__link" herf="#">Самозанятым</a>
            <a class="nav__link" herf="#">Малый и средний бизнес</a>
            <a class="nav__link" herf="#">Крупный бизнес</a>
           
        </nav>
        </div>
        </header>
</div>

</div>
<div class="about_img">
    <img src="C:/Users/op sky/PycharmProjects/server2/65.jpg" alt="">

</div>
</div>
</div>
<div class="news">
<div class="news_item">
    <div class="about_img">
        <img src="C:/Users/op sky/PycharmProjects/server2/1.jpg " alt="">
    </div>
     <div class="about__text">Подробнее</div>    
</div>
<div class="news">
    <div class="news_item">
        <div class="about_img">
            <img src="C:/Users/op sky/PycharmProjects/server2/1.jpg " alt="">
        </div>
        <div class="about__text">Подробнее</div> 
    </div>

    <div class="news">
        <div class="news_item">
            <div class="about_img">
                <img src="C:/Users/op sky/PycharmProjects/server2/1.jpg " alt="">
            </div>
            <div class="about__text">Подробнее</div> 
        </div>

</div>
</section>





CSS





body {
    font-family: 'Times New Romain', Helvetica, sans-serif;
   font-size: 16px; 
   color: rgb(66, 4, 251);
}
*,
*::before,
*::after{
    box-sizing:border-box;
}

h1,h2,h3,h4,h5,h6,body {
    margin:0
}
/* conteiner*/
.container {
    width: 100%;
    max-width: 1000px;
    margin-top: 100px;
    margin: 0 auto;
    height: 0 auto;

}
/*intro*/
.intro {
    width: 100%;
    height: 50vh;

    background: url("C:/Users/op sky/PycharmProjects/server2") center no-repeat;
    background-size: cover;
    background-size: cover;
}
.intro__inner{
    width: 100%;
    max-width: 500px;
    margin: 250px;
    text-align: center ;
}
.intro__title 
{
    color: black;
font-size:45px;
font-weight: 100;
text-transform: uppercase;
text-align: center;
line-height: 2;
}
.intro__title::after{
    content: "";
    display: block;
    width: 60px;
    height:1px;
    margin: 60px auto 60px;
}

/*Header*/
.header {
    width: 100%;
    
    position: absolute;
    top: 0px;
    left: 100px;
    right:0;
z-index: 1000;
}
.header_inner {
    display: flex; 
    justify-content: space-between;
    align-items:center;

}
.header__logo {
    margin: 0px -30px;
    font-size: 30px;
    font-weight: 700px;
    color:blue;
}

/*Nav*/
.nav {
    font-size: 15px; /*частные лица и тд*/
    text-transform: uppercase;
}
.nav__link {
    display: inline-block;
    vertical-align: top;
    margin: -35px 105px;
    color:rgb(7, 10, 13);
    text-decoration: none;
    transition: color 0.4s linear;
}
.nav__link:after
{
 content: "";
 display: block;
 height: 2px;
 background-color: #fff;
 position: absolute;
}
.nav__link:hover {
    color:rgb(44, 37, 235)

}
/*Button*/
.btn{
    display: inline-block;
    vertical-align: top;
    padding: 4px 15px;
    border: 4px solid cornflowerblue;
font-size: 14px;
font: weight 700px;
    color: black;
    text-transform: uppercase;
    text-decoration: none;
    transition: background 0.4s
    linear, color 0.4s linear ;
}
.btn:hover{
    background-color: rgb(100, 149, 237);
    color: black;
}
/*Section*/
.section{
    padding: 80px 0;

}

.section__header{
    width: 100%;
    max-width: 950px;
    margin: 0 auto 40PX;
text-align: center;
}
.section__title{
    margin-bottom:30px;
    padding-bottom: 0;
    text-align: center;
    font-size: 19px;
    color: black;
}
.section__suptitle{
    margin-bottom: 80px;
    padding-bottom: 0;
    text-align: center;
    font-size: 29px;
    color: black;

}
.section__text
{
    margin-bottom: 60px;
    padding-bottom: 0;
    text-align: center;
    font-size: 29px;
    color: black;
}

/*News*/
.news{
    display: flex;
    justify-content: space-between;

}
.news_item{
    width: 480px;
    position: relative;

}
.news_item:hover .about_img{
    transform: translate3d(-20px,-10px,0);
}

.news_item:hover .about_img img{
    opacity: 0.7;

}
.news_item:hover .about__text{
    opacity: 1;
}
.about_img{
    background: linear-gradient(to bottom,#5705fc,#28c9ff);
    transition: transform .2s linear;
}
.about_img img{
    transition: opacity .2s linear;
}

.about__text {
    width: 440px;
    font-size: 18px;
    color: rgb(255, 255, 255);
    text-transform: uppercase;
    font-weight: 700;
    text-align:center;
    opacity: 0;
    position:absolute;
    top:50%;
    left: 0;
    z-index: 2;
    transform: translate3d(0,-50%,0);
    transition: opacity .2s linear;
}
