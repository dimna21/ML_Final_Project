# Walmart store sales forecasting

# Timeseries მოდელები
### Data exploration and preprocessing
Timeseries მოდელების გამოკვლევისა და პრეპროცესინგის ნაწილში შემდეგი დაკვირვებებია ნაწარმოები თითოეული ცვლადისთვის (გრაფიკები მოცემულია ერთი მაღაზიის ჯამური გაყიდვებისთვის):

1. IsHoliday ცვლადის ნაცვლად თითოეული თარიღისთვის ამ კონკრეტული დღესასწაულის ცვლადია გამოყვანილი. მადლიერების დღისა და შობის დროს გაყიდვები ყველაზე მეტად იზრდება, superbowl-ს და labor day-ს კი შესამჩნევი, მაგრამ შედარებით ნაკლები ზრდა მოჰყვება.
<img width="1389" height="790" alt="Unknown-4" src="https://github.com/user-attachments/assets/9f9a5b3a-625a-4d19-ab60-87862cebc67f" />


2. CPI, Fuel price, unemployment rate და temperature ცვლადები ერთი შეხედვით გაყიდვების პატერნების შესახებ მკაფიო ინფორმაციას არ იძლევიან, მაგრამ ექსპერიმენტის ჩატარებისთვის ვტოვებთ.
<img width="1189" height="589" alt="Unknown" src="https://github.com/user-attachments/assets/bcc06d55-9e7b-4263-b181-80d049034399" />
<img width="1189" height="589" alt="Unknown-1" src="https://github.com/user-attachments/assets/0f47deff-5d5b-402f-9b4f-68bc460c3bf5" />
<img width="1189" height="589" alt="Unknown-2" src="https://github.com/user-attachments/assets/0f196fad-b440-4474-9395-050fe2def9cf" />
<img width="1189" height="589" alt="Unknown-3" src="https://github.com/user-attachments/assets/e082980c-cae6-4885-b2c8-5b3e22bb1872" />


3. ვინაიდან დღესასწაულების ცვლადები ყველაზე გამოსადეგი იყო, ორი შედარებით მძლავრი პრედიქტორისთვის დამატებულია days_to_{holiday} ცვლადები. როგორც მოსალოდნელია, ეს ცვლადები პიკს აღწევენ 0 მნიშვნელობისას (days_to_thanksgiving-ის შემთხვევაში პიკი -30 დღეზეა, რადგან ეს მომენტი შობას ემთხვევა). ამასთანავე, მოცემულ ცვლადში შენახულია ინფორმაცია დღესასწაულამდე და მის შემდეგ გაყიდვების ცვლილებაში. როდესაც days_to_{holiday} უარყოფითია, ეს მოდელს ანიშნებს, რომ დღესასწაულის შემდგომ პერიოდს ვუყურებთ, დადებითობის შემთხვევაში კი დღესასწაულის წინა დღეების მანიშნებელია. 
<img width="1143" height="790" alt="Unknown-5" src="https://github.com/user-attachments/assets/07a388c9-f560-4a48-bdd0-909336084d6a" />
<img width="1143" height="790" alt="Unknown-6" src="https://github.com/user-attachments/assets/db10b25d-0280-48f7-84a0-bc63f01e9e04" />



### ARIMA
ARIMA 3 პატამეტრისაგან შედგება:
1. AR - ავტო-რეგრესიული პარამეტრი p აკონკრეტებს იმას, თუ რამდენი კვირის მონაცემებს დააკვირდეს მოდელი წინა კვირებიდან. მოდელი ირჩევს წინა კვირების ტოპ p მონაცემს. კვირების რიგითობა განისაზღვრება ავტოკორელაციის მაღალი მნიშვნელობით, ანუ თუ მიმდინარე კვირის გაყიდვის კორელაციები წინა 5 კვირის გაყიდვებთან არის  (10, 3, -2, 4, -9), აირჩევა მოდულით უდიდესი p ცალი პარამეტრების ოპტიმიზაციისთვის.
2. I - Integrated პარამეტრი d განსაზღვრავს, თუ რა ხარისხის სხვაობა არის მეზობელ სემპლებს შორის, ანუ: d=0 ნიშნავს ორიგინალ მნიშვნელობებს (სტანდარტული ARMA), d=1 მათ შორის სხვაობებს, d=2 მოიაზრებს სხვაობების სხვაობებს და ა.შ.
3. MA - moving average პარამეტრი q აკონკრეტებს, თუ წინა რამდენი ნაბიჯის ერორზე დაფუძნებით გაასწოროს მოდელმა პარამეტრები.

ვინაიდან ARIMA-ს არ შეუძლია გარე ცვლადების მიღება, იგი მხოლოდ weekly_sales ცვლადზეა დატრენინგებული. ოპტიმალური იქნებოდა ყველა store-department წყვილისთვის საკუთარი მოდელის გაკეთება, მაგრამ ზოგ მათგანს არასაკმარისი რაოდენობის დატა აქვს. ასევე, ტესტ სეტში გვხვდება 11 store-dept წყვილი, რომლებზე დატაც საერთოდ არ გვაქვს მოცემული. ამის გამო, დატრეინებულია 45 ცალი storewise aggregated მოდელი, პრედიქციის დროს კი მთელი მაღაზიის გაყიდვები მრავლდება queried დეპარტამენტის გაყიდვების პროპორციაზე. ხსენებული 11 დეპარტამენტისთვის ბრუნდება მაღაზიის გაყიდვების საშუალო მნიშვნელობა.

ტრენინგის დროს მონაცემები დაყოფილია train/val <=> 123/20 კვირად. გატესტილია შემდეგი (p,q,d) პარამეტრები: [1,1,0], [0,1,1], [1,1,1],[2,1,1],[1,1,2],[2,1,2],[0,1,2],[3,1,1],[1,1,3], [4,1,1]. მათგან საუკეთესო აღმოჩნდა (2,1,1), რომელმაც kaggle-ს ტესტ სეტზე 4700 WMAE აიღო. 


### SARIMAX

### Prophet


# ხის მოდელები
### Data exploration and preprocessing


### XGBoost

### LightGBM


# ნეირონული ქსელები
### Data exploration and preprocessing

### N-BEATS

### Temporal Fusion Transformer

### PatchTST

### DLinear


