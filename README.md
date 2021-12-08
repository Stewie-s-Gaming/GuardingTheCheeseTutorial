# GuardingTheCheeseTutorial

בנינו Tutorial למשחק אשר מדריך את השחקן צעד-צעד על המשחק – רעיון מרכזי, אויבים, איך משחקים, תהליך מרכזי וכל זה תוך ליווי של השחקן עם הודעות טקסט המפרטות מה עליו לעשות כדי להתקדם במדריך ההתחלתי. 
המדריך שבנינו הוא אינטראקטיבי עם השחקן ולא מאפשר לו ללכת לאיבוד, לאורך כל התהליך קיימות תיבות טקסט וחצים המסבירים על הנושא, מה עליו לעשות ואיך עליו לעשות זאת.
המדריך מדגים את המשחק ואת הרעיון המרכזי שלו. על מנת לעשות זאת בנינו עולם (דו-מימדי – בהתאם למשחק שלנו), כמובן כחלק מהרעיון המרכזי של המשחק השחקן הוא זה שבונה את העולם בעזרת מלכודות ומכשולים אך עדיין קיים שלד שאותו אנו מספקים לשחקן, ובמדריך ההתחלתי הדגמנו זאת.

https://eliavamar.itch.io/gcheeseweek7

## פירוט על רכיבי ה - Tutorial

הרעיון המרכזי של המדריך היה לחלק את החלקים השונים של המדריך וליצור איזושהי יישות אשר תשלוט עליהם, היישות הזאת היא ה - TutorialManager.
למעשה זהו אובייקט (Singleton) אשר קובע איזה רכיב יופיע בפני השחקן ומתי, כל זה בהתאם להתנהלות השחקן.
לאחר שבנינו את האובייקט הנ"ל יצרנו תבנית של Tutorial אשר כל שאר הסקריפטים היו צריכים לרשת על מנת שה-TutorialManager יוכל לשלוט עליהם.

# TutorialManager

רכיב זה הוא רכיב סינגלטון (קיים רק Instance אחד ממנו) אשר שולט בכל רכיבי הסצנה, מציג את רכיבי המדריך בצורה מסודרת ובהתאם לקלט השחקן.

```
public class TutorialManager : MonoBehaviour
{
    public List<Tutorial> tutorialList = new List<Tutorial>();
    public string currentExplanation;
    private static TutorialManager instance;
    bool isActive;
    public static TutorialManager Instance
    {
        get
        {
            if (instance == null)
                instance = GameObject.FindObjectOfType<TutorialManager>();
            return instance;
        }
    }
    
```

ניתן לראות לפי משתני המחלקה כי האובייקט TurotialManager נוצר פעם אחת וכל מי שמנסה ליצור אותו או לקרוא לו יוכל לקבל רק את ה-Instance שלו.
כמו כן, ניתן לראות כי הוא מכיל מערך של אובייקטים מסוג Tutorial, למעשה הוא יריץ אותם לפי סדר קבוע מראש.
צורת כתיבה זו היא מונחת עצמים ומודולארית, ניתן להוציא ולהכניס רכיבים למדריך בצורה דינמית וקלה בלי להשפיע על שאר רכיבי הסצנה.

# ממשק ה - Tutorial

אובייקטים שאחראים על הצגת תהליך כלשהו בעבור השחקן חייבים לממש ממשק זה על מנת שה-TutorialManger ישלוט בהם ויריץ אותם.

להלן הממשק:
```
public class Tutorial : MonoBehaviour
{
    public int order;
    public string explanation;

    private void Awake()
    {
        TutorialManager.Instance.tutorialList.Add(this);
    }

    public virtual void checkIfHappened()
    {

    }
}
```
ניתן לראות כי בעת יצירתו האובייקט מוסיף את עצמו ל-Instance של TutorialManager ועל ידי כך מנהל המדריך יודע אליו רכיבים קיימים במדריך.
כמו כן קיים לכל אובייקט כזה שדה order אשר קובע מה הסדר שלו מבחינת הרצת האובייקטים, וכך מנהל המדריך יודע איזה אובייקט להריץ ומתי.
בנוסף, קיימת פונקציה וירטואלית אשר כל מי שיירש ממנו ייאלץ לממש, פוקנציה זו היא הפונקציה אשר ה-TutorialManager מריץ ועל ידי כך קובע איזה אובייקט רץ כרגע.
אובייקט שלא יממש אותה לא יוכל לרוץ.

קריאת הפונקציה הנ"ל מה-TutorialManager:
```

    // Update is called once per frame
    void Update()
    {
        if (isActive) 
        {
            if (!currentTutorial.gameObject.activeInHierarchy) currentTutorial.gameObject.SetActive(true);
            currentTutorial.checkIfHappened(); 
        }
    }
```
למעשה ה-TutorialManager מריץ בצורה בלתי פוסקת את הפונקציה של האובייקט Tutorial, ועל ידי כך דואג שהאובייקט שתורו לרוץ רץ עד אשר הוא מסיים את ייעודו ולאחר מכן
מפנה את מקומו לתהליך של אובייקט אחר.

דוגמה לתהליך ראשוני אשר מממש את הממשק Tutorial:
```
public class Tutorial_1 : Tutorial
{
    int currentMessage;
    [SerializeField] GameObject[] text;
    private void Awake()
    {
        currentMessage = 0;
        foreach(var item in text)
        {
            item.SetActive(false);
        }
    }

    public override void checkIfHappened()
    {
        if (Input.GetKeyDown(KeyCode.Return))
        {
            text[currentMessage].SetActive(false);
            currentMessage++;
        }
        if (currentMessage==text.Length) {
            TutorialManager.Instance.CompletedTutorial();
            return; 
        }
        text[currentMessage].SetActive(true);
    }
}
```
למעשה ניתן לראות כי כל התהליך שלו מתבצע ע"י קריאה לפונקציה checkIfHappened אשר ניתן להתייחס אליה כמו פונקציית Update.



## Animator

על מנת ליצור תהליך התחלתי היה עלינו להתעסק עם רכיב זה ולהכירו מקרוב, יצא לנו להגדיר אנימציות של אויבים ודמויות במשחק


![Animator_Enemy](https://user-images.githubusercontent.com/73071299/144158450-2989f8a4-6ffc-43ec-b9a7-f32a6e2d3c2e.PNG)

בתמונה ניתון לראות כיצד הגדרנו אנימציה לדמות מסוג אויב, אשר תשתנה בהתאם למצב שלו במשחק.
למשל כאשר האויב זז האנימציה תזוז ותיראה כאילו הוא באמת הולך, וכאשר הוא עוצר היא תבצע תנועות חוזרות במקום (Idle).
שינוי המצבים של האובייקט באנימציה נעשה באמצעות פרמטרים אשר מוגדרים בתוך ה-Animator.
אל פרמטרים אלו ניגשים באמצעות סקריפטים ומשנים אותם כך שיתאימו למצב שנמצא בו האובייקט כרגע, למשל כאשר האובייקט זז
אז יש לשנות את פרמטר ה-Speed שהגדרנו בתמונה, וכאשר הוא זז ספציפית למעלה יש להגדיר את הפרמטר הבוליאני isUp כאמת.

למעשה עשינו זאת בעזרת הסקריפט הבא:

```
public class AnimateController : MonoBehaviour
{
    [SerializeField] Animator animator;
    float speed;
    bool up, down, sides;

    // Update is called once per frame
    void Awake()
    {
        speed = 0;
        up = down = sides = false;
    }
    public void setSpeed(float s)
    {
        speed = s;
    }
    public void setUp(bool b)
    {
        up = b;
    }
    public void setDown(bool b)
    {
        down = b;
    }
    public void setSides(bool b)
    {
        sides = b;
    }
    void Update()
    {
        animator.SetFloat("Speed", System.Math.Abs(speed));
        animator.SetBool("Up", up);
        animator.SetBool("Down", down);
        animator.SetBool("Sides",sides);
    }
}
```

על סקריפט זה "הרכבנו" את אובייקט ה-Animator של דמות האויב ועל ידי כך ניגשנו לפרמטרים שבתוך ה-Animator ושינינו אותם בהתאם למצב שלו.


