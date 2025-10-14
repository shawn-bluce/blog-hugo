---
title: "假数据制造机：Python 中的 Faker 库"
slug: "python-faker-library"
date: "2024-06-19T14:05:00+0000"
lastmod: "2025-01-17T03:22:32+0000"
draft: false
tags:
  - "Python"
  - "faker"
visibility: "public"
---
# 0X00 来骗，来偷袭

这次来介绍一个来骗来偷袭的 Python 库：Faker。我们平时经常会跟数据库、跟 csv 这些东西打交道。尤其是当你设计一个数据库表的时候，开发和测试环境中只有空荡荡一个表，没有测试数据就很尴尬。

Faker 就是设计来解决这种问题的，它可以快速生成各种你需要的假数据。安装和使用都非常简单：`pip install Faker` 就可以完成安装。

> 这篇 mini 博客的目的是解决「不知道自己不知道」的问题，也就是说明有这么一个库可以做什么，然后介绍简单的用法；具体这个库的完整用法还是要去查看文档。

这里给出一个简单的使用实例：

```python
    #!/usr/bin/env python3


    from faker import Faker


    faker = Faker('zh-cn')

    print(faker.name())
    print(faker.email())
    print(faker.ipv4())
    print(faker.ipv4_private())
    print(faker.ipv4_public())
    print(faker.city_name())
```

简单示例
```
OUTPUT:


    李林
    chao83@example.com
    96.124.160.192
    192.168.72.158
    220.137.146.132
    张家港
```

这里仅仅有两条需要注意的：

  1. 实例化 faker 的时候记得标记语言，默认是英文信息；
  2. 实例化的 faker 每次调用都会生成新的假数据，只需要实例化一次即可；

> Faker 支持生成非常非常非常多数据类型，这里就不也没必要一个个介绍出来。分享一个我自己的用法：可以用 `dir(faker)` 的方式看它究竟有多少假数据类型可用，也可以在 `iPython` 中实例化一个 `faker` 出来然后通过 `faker.` TAB 的方式进行补全，比如你输入 `faker.ip` TAB 就可以看到`ipv4()/ipv4_network_class()/ipv4_private()/ipve_public()/ipv6()`这些。

👇下面我让 GPT 生成了一段脚本，可以看到使用 Faker 创建假数据使用 AI 偷懒是多么的舒爽

```python
    #!/usr/bin/env python3

    import csv
    from faker import Faker

    # Set up Faker with Chinese locale
    fake = Faker('zh_CN')

    # Number of students to generate
    num_students = 100

    # List to store student data
    students_data = []

    # Generate data for 100 students
    for _ in range(num_students):
        student = {
            "name": fake.name(),
            "age": fake.random_int(min=18, max=25),
            "phone_number": fake.phone_number(),
            "email": fake.email(),
            "address": fake.address(),
            "student_id": fake.random_number(digits=8, fix_len=True)
        }
        students_data.append(student)

    # Define the CSV file name
    csv_file = 'students_info.csv'

    # Define the CSV header
    csv_columns = ["name", "age", "phone_number", "email", "address", "student_id"]

    # Write data to CSV file
    try:
        with open(csv_file, mode='w', newline='', encoding='utf-8') as csvfile:
            writer = csv.DictWriter(csvfile, fieldnames=csv_columns)
            writer.writeheader()
            writer.writerows(students_data)
            print(f'Data written to {csv_file} successfully.')
    except IOError:
        print("I/O error occurred when writing to the CSV file.")
```

生成出来的文件如下，乍一看是不是还挺像那么回事的

```
    name,age,phone_number,email,address,student_id
    杨文,21,15651446876,xiangjing@example.net,天津市东莞县沈北新西宁街T座 528464,38652303
    任莹,20,13151815142,ming06@example.org,澳门特别行政区海门县静安谭街E座 169654,62640223
    马斌,22,15317418861,weiguiying@example.com,山西省西安市南溪海门路c座 632185,51262627
    万龙,25,18675244909,pqiao@example.org,澳门特别行政区惠州县海港陈街u座 788331,12862421
    唐莹,21,14524896623,wli@example.org,天津市马鞍山市黄浦拉萨街w座 346616,13890353
    王秀芳,24,15093668582,jie32@example.net,辽宁省佛山县魏都何街x座 791424,75220986
    李建平,21,18293878743,xiulanzhong@example.com,香港特别行政区彬市崇文北京街F座 134329,11422454
    谢丽娟,19,13242202418,pqian@example.com,新疆维吾尔自治区香港县滨城南宁路w座 529098,18889078
    薛婷,20,13807465629,zliang@example.net,北京市云市花溪刘街e座 773539,35816274
    雷丽娟,18,18165303861,yong51@example.net,湖北省济南县孝南陈街J座 484354,27706654
    朱秀兰,19,15036330967,caogang@example.com,台湾省佳县城北荆门街K座 131039,16335754
    张丽丽,24,15032876465,hanxiuying@example.org,澳门特别行政区潜江市浔阳海口路j座 179700,60603103
    赵桂兰,20,13009006486,minxie@example.com,四川省桂芝市海陵李街s座 535125,70265537
    赵春梅,24,15251434633,xiaojie@example.org,北京市南昌市朝阳嘉禾街Q座 111575,84861999
    蓝玉英,23,15006072574,xiuyingbai@example.net,辽宁省齐齐哈尔市东丽张路X座 952846,59067909
    马超,22,15023489681,ishen@example.net,山东省广州市永川任路I座 246768,57163888
    冯丽丽,19,18941560722,suyong@example.org,重庆市辉县怀柔大冶街n座 651829,48840449
    张辉,22,15525627747,edu@example.org,内蒙古自治区南京市翔安林路Y座 828410,27849301
    龙帅,20,13044861787,gangchen@example.com,福建省波县城北辽阳街B座 699463,93809855
    王秀兰,23,15123952816,liyao@example.com,重庆市海燕市永川李路U座 589932,40837105
    何秀云,22,13389508938,guojuan@example.org,宁夏回族自治区坤市华龙长沙街A座 448755,16331208
    郑建,21,14773507926,na97@example.org,湖北省北镇市牧野邯郸街Q座 441383,52901105
    黄桂香,20,15983226045,xia34@example.org,江西省长沙县西峰郑街v座 555389,38532397
    雷桂花,21,15779091442,fli@example.org,青海省重庆县魏都金路r座 438666,25582281
    洪伟,24,15034551309,bhu@example.org,甘肃省淑华市长寿沈阳街D座 781857,51371004
    侯秀荣,20,15658144733,mindeng@example.org,河南省拉萨市兴山周路q座 690193,34181753
    霍丽娟,23,14755592995,tianjie@example.org,湖北省沈阳市怀柔香港街E座 798600,59323935
    李晶,21,14789714084,luomin@example.net,湖南省北京县牧野吴街m座 527079,20443595
    何莹,20,15371237169,mwang@example.com,云南省北镇市涪城东莞路m座 835686,31431046
    冯小红,20,13929478423,na46@example.net,安徽省春梅市华龙合山路x座 783485,42282144
    王婷婷,25,18953524230,jieduan@example.net,江西省芳市城东马鞍山街H座 846377,89518293
    萧秀荣,21,15764734584,uzhang@example.com,河南省嘉禾县普陀杨路m座 946236,60617348
    卫利,22,13443120525,dujuan@example.com,宁夏回族自治区拉萨市兴山汤路U座 430226,19339491
    陈凤兰,23,15701289189,xduan@example.com,吉林省永安市吉区陈街s座 321158,51780743
    吴琳,18,14529375646,juanjia@example.com,陕西省佳县新城刘路H座 759718,89498195
    叶波,24,18061561625,xialin@example.net,西藏自治区丽丽县友好潜江街s座 970715,75555133
    赵建国,19,15825328065,yan22@example.org,青海省武汉市南长澳门路t座 428291,85520541
    刘瑞,24,15266286662,wei75@example.com,甘肃省平县翔安潮州路E座 566121,33484433
    李阳,18,15814354022,ohuang@example.com,陕西省艳市兴山柳州路W座 810774,77096319
    乔志强,22,15310103951,tao51@example.com,澳门特别行政区林市锡山丁路X座 168869,12169760
    吴想,22,13234314473,gang25@example.com,北京市海口县沙市宋街m座 592337,54436277
    潘婷婷,22,18978526383,dliao@example.net,台湾省东莞县丰都胡街J座 709057,91807647
    姚婷,21,18956219835,yuanjuan@example.com,香港特别行政区斌县孝南张街O座 130247,12811809
    汪峰,22,13315098113,ufan@example.net,河北省建平市沈北新济南路m座 170271,49070156
    李丹,23,13476665953,xiuyingxie@example.org,辽宁省大冶市吉区海门路J座 372191,70769981
    刘辉,18,15140639015,yanfeng@example.net,湖南省香港县合川尹路C座 922526,19022713
    赵雷,21,13090100149,chaoqian@example.net,北京市春梅县静安袁路W座 362649,43043458
    胡莉,25,18974156576,wangang@example.org,江苏省彬县沙市蒋街H座 208132,25741533
    冯欢,24,14777889865,yong75@example.org,陕西省太原县东城何街T座 632289,17757971
    萧楠,18,14520094716,hzhu@example.org,天津市秀兰县翔安刘街E座 388636,12886631
    万建平,19,15915290360,weima@example.org,新疆维吾尔自治区秀芳市城北贵阳路f座 220049,48199977
    张杨,24,15633549143,xia00@example.org,贵州省马鞍山市浔阳曹路b座 183259,86763091
    王伟,25,18102638719,pingzhou@example.com,云南省娟县高明太原路b座 241294,47232715
    孙桂荣,24,13214329937,dingchao@example.com,浙江省柳州县上街贾路g座 637845,57010965
    胡艳,25,14551404458,fengxiulan@example.net,宁夏回族自治区秀芳县朝阳辛集街Y座 373194,99124759
    陈桂兰,22,15705819325,ama@example.com,宁夏回族自治区雷市锡山拉萨街T座 109340,60143910
    傅帅,18,13037296868,xiuying11@example.net,山东省勇县浔阳西安街B座 734661,81970997
    张斌,25,15919070157,mingmeng@example.net,四川省斌县翔安徐街P座 873821,45687729
    吕军,21,15004718726,renjie@example.org,贵州省武汉县淄川郑州街c座 878856,95639249
    刘建平,18,15325768175,pcao@example.net,湖南省潮州县魏都永安街C座 573040,90720683
    汪桂芝,24,13320503412,zhan@example.com,黑龙江省长沙县白云武汉路D座 459772,65769168
    罗玲,24,18084432364,axiong@example.com,海南省张家港市海陵羊路N座 678692,53135397
    郝玉华,25,15038955596,shiyan@example.com,贵州省丽娟市沙市邱街j座 419562,95848846
    庞玉英,22,13728364375,ping58@example.net,新疆维吾尔自治区彬市高明王街d座 648000,64832283
    谢刚,20,13006216710,maoli@example.org,宁夏回族自治区武汉市静安高街N座 984048,66413173
    黄秀梅,24,18219872373,zengguiying@example.com,河南省淑珍市白云哈尔滨路R座 365763,90582516
    牟华,22,13473653213,gang06@example.net,四川省张家港市海陵张路W座 759909,96673716
    王玉英,20,13267038755,guiyingqiu@example.net,西藏自治区沈阳县牧野拉萨路c座 477448,11364274
    王玲,24,13609736441,exiang@example.org,香港特别行政区桂英市涪城姚街p座 415052,30258177
    邢桂兰,25,13104159794,taogao@example.net,宁夏回族自治区柳州县龙潭刘路H座 233088,29134578
    张亮,19,13946106449,bjiang@example.com,云南省辛集市清河李街i座 135971,78324297
    胡秀芳,23,18713925682,tmao@example.net,宁夏回族自治区马鞍山县牧野吴路y座 268410,96802279
    李洁,19,18765278454,xiangfang@example.com,福建省台北市长寿汕尾路o座 347366,85574490
    蒋兵,24,13400202150,chao51@example.org,台湾省关岭市新城辽阳街B座 241442,90031700
    张利,24,18090151698,qinyang@example.org,海南省重庆县清城辛集街P座 162853,15567649
    柳瑜,23,15575643433,na47@example.org,西藏自治区合肥县江北黄街Z座 356756,48715558
    袁建军,21,15243210479,songxia@example.org,山东省欣市花溪赵街L座 841242,51808908
    王秀英,24,13353152189,syang@example.org,内蒙古自治区明市永川海门街N座 900031,11089589
    徐峰,20,13931311017,pengyang@example.net,辽宁省合肥县友好林路v座 702291,39487119
    傅建军,25,15568950307,leijuan@example.com,云南省晶市闵行李街w座 387061,58657816
    温英,23,13600204215,gongli@example.net,云南省长沙县大兴重庆路A座 172765,79112040
    王浩,24,18878254393,panyan@example.org,上海市亮市魏都孙路z座 173852,89935907
    苏婷婷,25,15768731314,hdeng@example.org,湖南省博市高港台北路H座 384664,86207148
    张晨,25,14574088802,chaoma@example.com,西藏自治区哈尔滨市大东任路T座 331900,36313643
    朱鑫,25,14786084196,nazhong@example.net,内蒙古自治区文县秀英王路x座 316251,43000433
    葛宁,21,13628124785,fyin@example.com,天津市涛市合川巢湖路I座 258119,13636102
    欧建华,22,13474462477,gkang@example.net,黑龙江省梅市吉区李路Q座 346143,40699001
    周淑英,22,15394468081,ming17@example.com,黑龙江省兴城县兴山张街x座 692324,66822153
    樊凯,18,15961375610,rhu@example.com,江西省太原市合川澳门路S座 134314,47105810
    杨莉,24,14535546194,li91@example.net,辽宁省旭县东城姚街U座 534685,47600036
    宫玉珍,22,13536666673,xiaqiang@example.org,吉林省太原县山亭罗路g座 409209,61974123
    顾鹏,25,18199286199,kongping@example.com,陕西省宁市浔阳贵阳路n座 403156,29450095
    井芳,18,18600909935,azhu@example.com,江苏省嘉禾市清河张路g座 619941,69174743
    宋玉华,20,18932355002,yjia@example.org,青海省澳门市海港程路U座 652351,57624414
    张建平,22,18155187736,wangming@example.com,湖南省莉市淄川呼和浩特路A座 964393,80044659
    姜斌,24,18803574186,tao82@example.org,山西省辛集市东城胡路v座 119458,25854974
    陈颖,23,14517412603,luowei@example.org,云南省建华市魏都徐街D座 571366,43305147
    李海燕,22,15193868140,yangzhao@example.org,湖北省石家庄市丰都台北路T座 551877,56451677
    李桂荣,20,15616840094,taoguo@example.com,上海市拉萨市大东叶街d座 577821,31959883
    张婷婷,22,13166077906,li41@example.com,西藏自治区永安市永川范路i座 943614,66647808
```