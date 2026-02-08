# goldenrouth
מערכת לזיהוי איומים אוויריים
על הפרויקט
מערכת זו נועדה לניהול והגנה על נקודה גיאוגרפית מוגדרת מפני איומים אוויריים בזמן אמת. המערכת מתחברת לשרתי ה-API של OpenSky, שולפת נתוני טיסה חיים, ומנתחת מי מהמטוסים מהווה איום על הנכס בהתאם לרדיוס שהוגדר.

המערכת לא רק מחשבת מרחק, אלא מבצעת ניתוח וקטורי של מהירות המטוס ומחשבת מסלולי עקיפה במידה וישנם "כיפות ברזל" (מכשולים) בדרך.



הוראות הרצה
הפרויקט בנוי בצורה של Full Docker, מה שאומר שאין צורך להתקין מסדי נתונים או ספריות באופן מקומי. הכל קורה בתוך הקובץ.

דרישות קדם:
מותקן Docker Desktop על המחשב והוא דולק.

שלבי הפעלה:
הורדת הפרויקט: חלצו את קובץ ה-ZIP לתיקייה על המחשב.

בנייה והרצה: פתחו טרמינל (CMD) בתיקייה הראשית של הפרויקט והריצו:


                                                                                                                       docker-compose up --build -d
פקודה זו תבנה את הממשק, השרת ומסד הנתונים ותפעיל אותם ברקע.

סנכרון מסד הנתונים (Prisma): לאחר שהפרוייקט עלה, יש להריץ את הפקודה הבאה כדי ליצור את הטבלאות ב-Database:


                                                                                  docker exec -it golden_server npx prisma migrate dev --name init
גישה לממשק: פתחו את הדפדפן בכתובת: http://localhost:3000
 




שימוש בממשק המשתמש
הזנת קורדינאטות: הזינו קואורדינטות (קו רוחב וקו אורך), מהירות ורדיוס האיום.

מערכת הגנה: ניתן להגדיר נקודה שלישית המהווה מכשול. המערכת תחשב את זמן ההגעה בהתחשב בכך שהמטוס חייב לעקוף את המעגל הזה.

תוצאות בזמן אמת: המערכת תציג את המטוס המאיים ביותר, מרחקו, וזמן הגעתו המשוער.

שמירה וייבוא: ניתן לשמור מבצעים לתוך מסד הנתונים ולייבא אותם מאוחר יותר דרך כפתור "ייבוא מבצע שמור".

הסבר על הודעת "לא ניתן לחישוב":
במידה ומופיעה ההודעה "זמן סגירה משוקלל: לא ניתן לחישוב", המשמעות היא שהמערכת זיהתה מטוס בטווח האיום, אך הנתונים שמגיעים מה-API של OpenSky לגבי אותו מטוס ספציפי הם חלקיים (למשל: חסרה מהירות או חסר כיוון טיסה מדויק), ולכן לא ניתן לבצע את החישוב הווקטורי.




הגבלות API והחלטות תכנוניות
המערכת משתמשת ב-OpenSky API בגרסה החינמית, המגבילה את כמות הבקשות שניתן לשלוח בדקה.

התנהגות השרת: במידה ותבצעו שינויים מהירים מדי בטופס, ה-API עלול לחסום את הבקשות לזמן קצר, וזה עלול להיראות כאילו השרת "קרוס". בפועל, זוהי חסימה חיצונית.

החלטת "זמן אמת": חשבתי על פתרון של שמירת הנתונים (Caching) ומחיקתם כל 10 שניות כדי להקל על ה-API, אך החלטתי לוותר על כך. במערכת לזיהוי איומים, דיוק של "זמן אמת" הוא קריטי, ושמירת נתונים ישנים (אפילו ל-10 שניות) פוגעת במקצועיות המערכת ובבטיחות הנתונים.







Tech Stack

Tech Stack
Frontend: React.js Building a dynamic user interface that enables real-time updates of calculation results and maps without requiring a page refresh. The use of React Hooks (such as useState and useEffect) allows for efficient state management of user inputs and seamless data flow from the server.

Backend: Node.js + Express An asynchronous runtime environment that efficiently handles concurrent requests, such as fetching data from external APIs and communicating with the database. Express is used to manage the API endpoints that perform the complex logic and vector calculations.

Database: PostgreSQL A powerful relational database used to store operation history in a structured and reliable manner. It enables the storage of complex objects (such as aircraft data) and allows for fast retrieval based on date or unique identifiers.

ORM: Prisma A modern database toolkit (ORM) that allows interacting with the database using JavaScript objects instead of writing manual SQL queries. Prisma ensures full synchronization between the code structure and the database tables within Docker, maintaining data integrity.

Containerization: Docker & Docker Compose Utilizing containers ensures that the project runs identically on any machine, independent of local software installations. Docker Compose allows the server, interface, and database to be launched with a single simple command, simplifying deployment and testing.

Libraries: Geolib A mathematical library for geographic calculations that accounts for the Earth's curvature (Great Circle). It provides the precision required for calculating distances, bearings, and radii—the core foundations of the system's vector threat analysis.



פירוט מתמטי של סעיפי הבונוס 
סעיף ד': זמן סגירה ווקטורי
במקום להסתכל רק על המהירות הכללית של המטוס, המערכת בוחנת כמה מהמהירות הזו באמת "מקדמת" אותו אלינו.

מציאת כיוון המטרה: קודם כל, המערכת מחשבת את הזווית המדויקת שבין המטוס לבין המיקום שלנו.

חישוב הפרש הזוויות: המערכת בודקת את ההפרש בין כיוון הטיסה הממשי של המטוס לבין הזווית שמובילה אלינו.

היטל המהירות: בעזרת פונקציית קוסינוס, המערכת מחשבת איזה "חלק" מהמהירות של המטוס פונה ישירות לעברנו. אם הוא טס הצידה, רק חלק מהמהירות נחשב אם הוא טס ישירות אלינו, כל המהירות נחשבת.

מהירות סגירה כוללת: המערכת מחברת את רכיב המהירות של המטוס עם מהירות התנועה של הנכס עצמו (אם הוא בתנועה), כדי לקבל את מהירות ההתקרבות האמיתית ביניהם.

חישוב הזמן: לבסוף, המערכת מחלקת את המרחק שנשאר במהירות הסגירה שחושבה, כדי לקבל זמן הגעה מדויק בשעות.

סעיף ה': עקיפת מכשולים 
כאשר קיים אזור הגנה בדרך, המטוס לא יכול לטוס בקו ישר. המערכת מחשבת את הדרך העוקפת הקצרה ביותר:

בדיקת חסימה: המערכת בודקת האם הקו הישר שבין המטוס לנכס חותך את המעגל של אזור ההגנה.

חישוב זוויות המשיקים: המערכת מחשבת את הזוויות שבהן המטוס והמיקום שהכנסנו צריכים להסתכל כדי לפגוע בדיוק בשולי המעגל. זה נעשה על ידי חישוב היחס בין רדיוס המעגל למרחק מהמרכז.

חישוב המשיקים: בעזרת משפט פיתגורס, המערכת מוצאת את אורך הקו הישר מהמטוס עד לנקודה שבה הוא נוגע במעגל, ומהנכס עד לנקודה המקבילה בצד השני.

חישוב זווית הקשת: המערכת מחשבת כמה המטוס צריך להסתובב על שפת המעגל עצמו כדי לעקוף אותו. הזווית הזו היא ההפרש שנותר מהזווית הכוללת לאחר החסרת זוויות המשיקים.

חישוב המסלול הכולל: אורך המסלול החדש הוא סכום של שלושה חלקים: הקו הישר מהמטוס למעגל, הקו הישר מהמעגל לנכס, ואורך הקשת שביניהם (הרדיוס כפול זווית הקשת).
