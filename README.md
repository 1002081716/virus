# virus
import React, { useState, useEffect, useRef } from 'react';
import { BookOpen, BrainCircuit, CheckSquare, ChevronLeft, ChevronRight, Activity, ShieldAlert, Dna, Image as ImageIcon, Youtube, MessageCircle, Sparkles, Loader2, X, Send } from 'lucide-react';

// --- GEMINI API SETUP ---
const apiKey = ""; // The execution environment provides the key at runtime

const callGeminiAPI = async (prompt, systemInstruction = "", jsonSchema = null) => {
  const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
  
  const payload = {
    contents: [{ parts: [{ text: prompt }] }],
    systemInstruction: { parts: [{ text: systemInstruction }] },
    generationConfig: {}
  };

  if (jsonSchema) {
    payload.generationConfig.responseMimeType = "application/json";
    payload.generationConfig.responseSchema = jsonSchema;
  }

  const retries = [1000, 2000, 4000, 8000, 16000];
  
  for (let i = 0; i < retries.length; i++) {
    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });
      
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      
      const data = await response.json();
      const textResponse = data.candidates?.[0]?.content?.parts?.[0]?.text;
      
      return jsonSchema ? JSON.parse(textResponse) : textResponse;
    } catch (error) {
      if (i === retries.length - 1) {
        console.error("Gemini API Error:", error);
        throw error;
      }
      await new Promise(r => setTimeout(r, retries[i]));
    }
  }
};

// --- DATA STRUCTURE ---
const learningData = {
  intro: {
    id: 'intro',
    title: 'מבוא: מהם נגיפים?',
    icon: <Activity className="w-6 h-6" />,
    poster: { 
      title: "מבנה הנגיף ומחזור חייו", 
      image: <img src="unnamed (20).jpg" alt="מבנה הנגיף ומחזור חייו" className="max-w-full max-h-[60vh] object-contain rounded-md" /> 
    },
    video: { url: "https://www.youtube.com/embed/8FqlTslU22s", title: "חדירת נגיף לתא (אנימציה)" },
    theory: [
      { text: "נגיף (וירוס) הוא {טפיל מוחלט} בגודל מיקרוסקופי. הוא אינו מסוגל להתרבות או לקיים תהליכי חיים מחוץ ל{תא מאכסן} (פונדקאי)." },
      { text: "המבנה הבסיסי של נגיף כולל חומר תורשתי ({DNA או RNA}) העטוף במעטפת חלבונית הנקראת {קפסיד}. לחלק מהנגיפים יש גם מעטפת שומנית חיצונית." },
      { text: "מחזור חיי הנגיף כולל 5 שלבים עיקריים: 1. {היצמדות} לקולטנים ייחודיים בתא המאכסן. 2. {חדירה} לתוך התא. 3. {שכפול} החומר התורשתי וייצור חלבוני הנגיף על ידי מערכות התא. 4. {הרכבת} נגיפים חדשים. 5. {שחרור} הנגיפים מהתא (לעיתים תוך פיצוץ התא - ליזה, או בהנצה)." }
    ],
    mindmap: {
      root: "נגיף (וירוס)",
      branches: [
        { label: "מבנה", sub: ["חומר תורשתי (DNA/RNA)", "קפסיד (מעטפת חלבון)", "מעטפת שומנית (בחלקם)"] },
        { label: "מאפיינים", sub: ["טפיל תאי מוחלט", "ספציפיות לתא מאכסן", "אינו יצור חי עצמאי"] },
        { label: "מחזור חיים", sub: ["1. היצמדות", "2. חדירה", "3. שכפול וייצור", "4. הרכבה", "5. שחרור"] }
      ]
    },
    flashcards: [
      { front: "מהו קפסיד?", back: "מעטפת חלבונית המגינה על החומר התורשתי של הנגיף." },
      { front: "מדוע הנגיף נקרא 'טפיל מוחלט'?", back: "משום שאין לו מערכות ייצור אנרגיה או חלבונים משלו, והוא חייב לחדור לתא מאכסן כדי להתרבות." },
      { front: "מה קובע לאיזה תא הנגיף יחדור?", back: "התאמה ספציפית (כמו מפתח ומנעול) בין חלבונים במעטפת הנגיף לבין קולטנים על קרום התא המאכסן." }
    ],
    quiz: [
      {
        question: "איזה מהמשפטים הבאים נכון לגבי כל הנגיפים?",
        options: ["כולם מכילים DNA", "כולם מכילים מעטפת שומנית", "כולם טפילים מוחלטים", "כולם פוגעים בבני אדם"],
        correct: 2
      },
      {
        question: "מה קורה בשלב ה'היצמדות' במחזור חיי הנגיף?",
        options: ["הנגיף משתכפל בתוך התא", "חלבוני הנגיף נקשרים לקולטנים ספציפיים על קרום התא", "הנגיף מפוצץ את התא המאכסן", "התא המאכסן מייצר נוגדנים"],
        correct: 1
      },
      {
        question: "כיצד שונה נגיף (וירוס) מחיידק?",
        options: ["הוא מכיל חומר תורשתי", "הוא יכול להתרבות רק בתוך תא מאכסן", "הוא גורם למחלות", "יש לו קרום תא וציטופלזמה"],
        correct: 1
      },
      {
        question: "מהו המסלול הליטי במחזור החיים של נגיף (כגון בקטריופאג')?",
        options: ["הנגיף משתלב בגנום התא ונשאר רדום (ליזוגני)", "הנגיף מתרבה וגורם להרס (פיצוץ) התא המאכסן", "הנגיף מתרבה מחוץ לתא", "הנגיף עוזר לתא המאכסן להתחלק"],
        correct: 1
      }
    ]
  },
  herpes: {
    id: 'herpes',
    title: 'נגיף ה-DNA: הרפס',
    icon: <Dna className="w-6 h-6" />,
    poster: { 
      title: "הדבקה ומחזור חיים - נגיף ההרפס", 
      image: "" 
    },
    video: { url: "https://www.youtube.com/embed/Q2PEhXPaF8M", title: "מחזור החיים וההדבקה של נגיף ההרפס" },
    theory: [
      { text: "נגיף ההרפס (HSV-1) הוא נגיף המכיל חומר תורשתי מסוג {DNA}. הוא חודר בעיקר לתאי העור והריריות, למשל באזור השפתיים." },
      { text: "מחזור חיים: הנגיף מוכנס לתא. ה-DNA הנגיפי חודר ל{גרעין התא}. שם, ה-DNA הנגיפי עובר תהליכי שעתוק (על ידי האנזים RNA פולימראז של התא) ותרגום ליצירת חלבוני נגיף חדשים." },
      { text: "תכונה ייחודית: ההרפס יכול להיכנס ל{מצב רדום} (לטנטי) בתוך תאי עצב. במצב זה הוא אינו מתרבה ולא גורם לתסמינים. גירויים שונים (כמו לחץ או קרינת שמש) יכולים 'לעורר' אותו למחזור פעיל." },
      { text: "טיפול: התרופה המוכרת זובירקס (Acyclovir) פוגעת באנזים {DNA פולימראז הנגיפי}, ובכך עוצרת את שכפול הנגיף מבלי לפגוע בתאי הגוף." }
    ],
    mindmap: {
      root: "נגיף ההרפס (DNA)",
      branches: [
        { label: "חומר תורשתי", sub: ["DNA דו-גדילי", "משתכפל בגרעין התא המאכסן"] },
        { label: "מחזור מחלה", sub: ["הדבקה פעילה (פצעים בעור)", "מצב רדום (בתאי עצב)", "התפרצות חוזרת"] },
        { label: "טיפול", sub: ["תרופת האציקלוביר (זובירקס)", "מעכבת DNA פולימראז נגיפי"] }
      ]
    },
    flashcards: [
      { front: "היכן נמצא החומר התורשתי של ההרפס כשהוא במצב רדום?", back: "בתאי עצב." },
      { front: "כיצד פועלת התרופה נגד הרפס?", back: "היא מעכבת את האנזים DNA פולימראז של הנגיף, ובכך מונעת את שכפול החומר התורשתי שלו." },
      { front: "היכן מתרחש שכפול ה-DNA הנגיפי של ההרפס?", back: "בתוך גרעין התא המאכסן." }
    ],
    quiz: [
      {
        question: "מדוע אדם שנדבק בהרפס סובל מהתפרצויות חוזרות של המחלה?",
        options: ["כי הוא נדבק מחדש מאנשים אחרים", "כי הנגיף נשאר במצב רדום בתאי העצב ומתעורר מדי פעם", "כי הנגיף עובר מוטציות מהירות מאוד", "כי התרופות לא יעילות כלל"],
        correct: 1
      },
      {
        question: "איזה אנזים הנגיף מנצל מהתא המאכסן כדי לייצר RNA-שליח (mRNA)?",
        options: ["DNA פולימראז", "רוורס טרנסקריפטאז", "RNA פולימראז", "ליגאז"],
        correct: 2
      },
      {
        question: "נגיף ההרפס מאופיין במצב לטנטי (רדום). משמעות הדבר היא:",
        options: ["הנגיף מת מחוץ לגוף תוך זמן קצר", "החומר התורשתי נמצא בתא ואינו גורם לייצור נגיפים עד שיופעל", "הנגיף מתרבה בקצב מסוחרר והורס את התא מיידית", "הנגיף תוקף רק חיידקים"],
        correct: 1
      }
    ]
  },
  hiv: {
    id: 'hiv',
    title: 'רטרווירוס: HIV (איידס)',
    icon: <ShieldAlert className="w-6 h-6" />,
    poster: { 
      title: "מנגנון הפעולה של נגיף ה-HIV (איידס)", 
      image: "" 
    },
    video: { url: "https://www.youtube.com/embed/RO8MP3wMvqg", title: "מחזור חיי נגיף ה-HIV (אנימציה רפואית תלת-ממדית)" },
    theory: [
      { text: "נגיף ה-HIV הוא {רטרווירוס}. כלומר, החומר התורשתי שלו הוא מסוג {RNA}, וכאשר הוא חודר לתא הוא הופך אותו ל-DNA." },
      { text: "מחזור חיים: הנגיף תוקף באופן ספציפי את תאי מערכת החיסון הנקראים {תאי T מסייעים} (CD4). לאחר החדירה, אנזים ייחודי של הנגיף שנקרא {מתעתק במהופך} (Reverse Transcriptase) בונה DNA על בסיס ה-RNA הנגיפי." },
      { text: "ה-DNA הנגיפי חודר לגרעין ומשתלב ב-DNA של התא בעזרת האנזים {אינטגראז}. משלב זה התא מייצר נגיפי HIV חדשים, מה שמוביל בסופו של דבר להרס מערכת החיסון ולמחלת ה{איידס}." },
      { text: "טיפול: מכיוון שהנגיף עובר המון מוטציות, נותנים לחולים {קוקטייל תרופות} - שילוב של תרופות (כמו AZT) הפוגעות במקביל באנזימים שונים של הנגיף (כמו ה-RT והפרוטאז)." }
    ],
    mindmap: {
      root: "נגיף ה-HIV (רטרווירוס)",
      branches: [
        { label: "הדבקה", sub: ["מכיל RNA כחומר תורשתי", "פוגע בתאי T מסייעים במערכת החיסון"] },
        { label: "אנזימים ייחודיים", sub: ["מתעתק במהופך (RNA -> DNA)", "אינטגראז (שילוב בגנום)", "פרוטאז (חיתוך חלבונים)"] },
        { label: "טיפול", sub: ["קוקטייל תרופות", "מונע עמידות בגלל מוטציות רבות"] }
      ]
    },
    flashcards: [
      { front: "מהו התפקיד של האנזים 'מתעתק במהופך' (RT)?", back: "לבנות מולקולת DNA על בסיס תבנית ה-RNA של הנגיף, מיד לאחר חדירתו לתא." },
      { front: "אילו תאים נגיף ה-HIV תוקף, ומהי התוצאה?", back: "הוא תוקף תאי T מסייעים (תאי דם לבנים). התוצאה היא קריסה של מערכת החיסון (מחלת האיידס)." },
      { front: "מדוע חולי HIV מטופלים ב'קוקטייל' תרופות ולא בתרופה אחת?", back: "לנגיף ה-HIV יש קצב מוטציות גבוה מאוד. שילוב של מספר תרופות פוגע במספר מנגנונים במקביל ומקטין את הסיכוי של הנגיף לפתח עמידות." }
    ],
    quiz: [
      {
        question: "איזה מן התהליכים הבאים מתרחש במחזור החיים של נגיף ה-HIV אך לא במחזור החיים של נגיף ההרפס?",
        options: ["יצירת RNA שליח (mRNA) בגרעין", "שעתוק לאחור של RNA ל-DNA", "הרכבת קפסיד חלבוני", "יציאה מהתא באמצעות חתיכת קרום"],
        correct: 1
      },
      {
        question: "כיצד התרופה AZT פוגעת בנגיף ה-HIV?",
        options: ["היא הורסת את המעטפת השומנית שלו", "היא פוגעת באנזים המתעתק במהופך (RT)", "היא חוסמת את הקולטנים על פני תא ה-T", "היא מעודדת ייצור של תאי דם לבנים"],
        correct: 1
      },
      {
        question: "מהי הסיבה העיקרית לקושי בפיתוח חיסון נגד נגיף ה-HIV (וגם נגד נגיפי שפעת מסוימים)?",
        options: ["הנגיף קטן מדי ולכן מערכת החיסון לא מזהה אותו", "הנגיף משנה לעיתים קרובות את חלבוני המעטפת שלו עקב שיעור מוטציות גבוה", "הנגיף תוקף רק תאי עור שאינם נגישים לנוגדנים", "הנגיף אינו מכיל חומר תורשתי כלל"],
        correct: 1
      }
    ]
  }
};

// --- AI COMPONENTS ---

const AiChatBox = ({ currentTopic }) => {
  const [isOpen, setIsOpen] = useState(false);
  const [messages, setMessages] = useState([
    { role: 'model', text: 'היי! אני העוזר הווירטואלי שלך לביולוגיה. יש לך שאלה לגבי החומר?' }
  ]);
  const [input, setInput] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const messagesEndRef = useRef(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  useEffect(() => {
    if (isOpen) scrollToBottom();
  }, [messages, isOpen]);

  const handleSend = async () => {
    if (!input.trim() || isLoading) return;
    
    const userMsg = input.trim();
    setInput('');
    setMessages(prev => [...prev, { role: 'user', text: userMsg }]);
    setIsLoading(true);

    const systemInstruction = `אתה מורה ביולוגיה ידידותי בתיכון ישראלי. 
    התלמיד כרגע לומד על הנושא: "${currentTopic.title}".
    ענה על שאלות התלמיד בעברית בלבד. היה תמציתי, מעודד וברור. הסבר מושגים מסובכים בפשטות.`;

    try {
      const responseText = await callGeminiAPI(userMsg, systemInstruction);
      setMessages(prev => [...prev, { role: 'model', text: responseText }]);
    } catch (error) {
      setMessages(prev => [...prev, { role: 'model', text: 'מצטער, חלה שגיאה בתקשורת. נסה שוב מאוחר יותר.' }]);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <>
      {!isOpen && (
        <button 
          onClick={() => setIsOpen(true)}
          className="fixed bottom-6 left-6 z-50 bg-indigo-600 text-white p-4 rounded-full shadow-2xl hover:bg-indigo-700 transition-transform transform hover:scale-110 flex items-center justify-center group"
          title="שאל את המורה הווירטואלי"
        >
          <Sparkles className="absolute -top-1 -right-1 w-4 h-4 text-yellow-300" />
          <MessageCircle className="w-8 h-8" />
        </button>
      )}

      {isOpen && (
        <div className="fixed bottom-6 left-6 z-50 w-80 md:w-96 h-[500px] bg-white rounded-2xl shadow-2xl border border-gray-200 flex flex-col overflow-hidden animate-fade-in">
          <div className="bg-indigo-600 text-white p-4 flex justify-between items-center shadow-md z-10">
            <div className="flex items-center gap-2">
              <Sparkles className="w-5 h-5 text-yellow-300" />
              <h3 className="font-bold text-lg">המורה לביולוגיה (AI)</h3>
            </div>
            <button onClick={() => setIsOpen(false)} className="text-white hover:bg-indigo-500 p-1 rounded">
              <X className="w-5 h-5" />
            </button>
          </div>
          
          <div className="flex-1 overflow-y-auto p-4 bg-gray-50 space-y-4 flex flex-col">
            {messages.map((msg, idx) => (
              <div key={idx} className={`max-w-[85%] rounded-2xl p-3 text-sm shadow-sm ${
                msg.role === 'user' 
                  ? 'bg-blue-600 text-white self-end rounded-br-none' 
                  : 'bg-white text-gray-800 border border-gray-100 self-start rounded-bl-none'
              }`}>
                {msg.text}
              </div>
            ))}
            {isLoading && (
              <div className="bg-white text-gray-500 border border-gray-100 self-start rounded-2xl rounded-bl-none p-3 shadow-sm flex items-center gap-2">
                <Loader2 className="w-4 h-4 animate-spin text-indigo-500" />
                <span className="text-sm">מקליד תשוּבה...</span>
              </div>
            )}
            <div ref={messagesEndRef} />
          </div>

          <div className="p-3 bg-white border-t border-gray-200">
            <div className="relative flex items-center">
              <input 
                type="text" 
                value={input}
                onChange={e => setInput(e.target.value)}
                onKeyDown={e => e.key === 'Enter' && handleSend()}
                placeholder="שאל אותי משהו על נגיפים..."
                className="w-full bg-gray-100 border-none rounded-full py-3 px-4 pr-12 text-sm focus:ring-2 focus:ring-indigo-500 outline-none"
              />
              <button 
                onClick={handleSend}
                disabled={isLoading || !input.trim()}
                className="absolute right-2 p-2 text-indigo-600 hover:bg-indigo-100 rounded-full disabled:text-gray-400 transition-colors"
              >
                <Send className="w-5 h-5" />
              </button>
            </div>
          </div>
        </div>
      )}
    </>
  );
};


// --- REGULAR COMPONENTS ---

const PosterViewer = ({ poster }) => {
  if (!poster) return null;
  return (
    <div className="bg-white rounded-xl shadow-md border border-gray-200 overflow-hidden mb-8">
      <div className="bg-gradient-to-r from-blue-600 to-indigo-600 text-white p-3 flex items-center gap-2">
        <ImageIcon className="w-5 h-5" />
        <h3 className="font-bold">פוסטר למידה: {poster.title}</h3>
      </div>
      <div className="p-6 bg-gray-50 flex justify-center items-center">
        <div className="w-full max-w-3xl aspect-[4/3] bg-white border-4 border-gray-200 rounded-lg shadow-sm flex items-center justify-center text-gray-500 font-bold text-xl text-center p-4">
          {poster.image}
        </div>
      </div>
    </div>
  );
};

const VideoPlayer = ({ video }) => {
  if (!video) return null;
  return (
    <div className="bg-white rounded-xl shadow-md border border-gray-200 overflow-hidden mb-8">
      <div className="bg-gradient-to-r from-red-600 to-red-700 text-white p-3 flex items-center gap-2">
        <Youtube className="w-5 h-5" />
        <h3 className="font-bold">סרטון העשרה: {video.title}</h3>
      </div>
      <div className="p-0 aspect-video w-full bg-black">
        <iframe 
          className="w-full h-full" 
          src={video.url} 
          title={video.title}
          frameBorder="0" 
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
          allowFullScreen
        ></iframe>
      </div>
    </div>
  );
};

const InteractiveText = ({ text }) => {
  const parts = text.split(/({[^}]+})/g);
  return (
    <p className="text-lg leading-relaxed mb-4">
      {parts.map((part, index) => {
        if (part.startsWith('{') && part.endsWith('}')) {
          const word = part.slice(1, -1);
          return <HiddenWord key={index} word={word} />;
        }
        return <span key={index}>{part}</span>;
      })}
    </p>
  );
};

const HiddenWord = ({ word }) => {
  const [revealed, setRevealed] = useState(false);
  return (
    <span 
      onClick={() => setRevealed(true)}
      className={`inline-block mx-1 px-2 py-1 rounded cursor-pointer transition-all duration-300 font-bold ${
        revealed 
          ? 'bg-blue-100 text-blue-800 border border-blue-300 shadow-sm' 
          : 'bg-gray-200 text-transparent border border-dashed border-gray-400 hover:bg-gray-300'
      }`}
      style={{ userSelect: revealed ? 'auto' : 'none' }}
      title={revealed ? '' : 'לחץ כדי לגלות'}
    >
      {revealed ? word : '???'}
    </span>
  );
};

const Flashcard = ({ front, back }) => {
  const [flipped, setFlipped] = useState(false);
  return (
    <div 
      className="relative w-full h-48 cursor-pointer perspective-1000 mb-4"
      onClick={() => setFlipped(!flipped)}
    >
      <div className={`w-full h-full transition-transform duration-500 transform-style-3d ${flipped ? 'rotate-y-180' : ''}`}>
        <div className="absolute w-full h-full backface-hidden bg-white border-2 border-indigo-200 rounded-xl shadow-md flex items-center justify-center p-6 text-center">
          <h3 className="text-xl font-bold text-indigo-900">{front}</h3>
          <p className="absolute bottom-2 text-xs text-indigo-400">לחץ כדי להפוך</p>
        </div>
        <div className="absolute w-full h-full backface-hidden rotate-y-180 bg-indigo-50 border-2 border-indigo-300 rounded-xl shadow-md flex items-center justify-center p-6 text-center">
          <p className="text-lg text-indigo-800">{back}</p>
        </div>
      </div>
    </div>
  );
};

const MindMap = ({ data }) => {
  return (
    <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-100 overflow-x-auto">
      <div className="flex flex-col items-center">
        <div className="bg-emerald-500 text-white font-bold py-3 px-6 rounded-lg shadow-md mb-8 z-10 relative">
          {data.root}
          <div className="absolute h-8 w-1 bg-gray-300 left-1/2 -bottom-8 transform -translate-x-1/2"></div>
        </div>
        <div className="w-3/4 h-1 bg-gray-300 flex justify-between relative mb-8">
          {data.branches.map((_, i) => (
             <div key={`vert-${i}`} className="h-8 w-1 bg-gray-300 absolute -top-1" style={{left: `${(i / (data.branches.length - 1)) * 100}%`}}></div>
          ))}
           {data.branches.map((_, i) => (
             <div key={`vert-down-${i}`} className="h-8 w-1 bg-gray-300 absolute top-0" style={{left: `${(i / (data.branches.length - 1)) * 100}%`}}></div>
          ))}
        </div>
        <div className="flex w-full justify-between gap-4">
          {data.branches.map((branch, index) => (
            <div key={index} className="flex-1 flex flex-col items-center min-w-[150px]">
              <div className="bg-emerald-100 border border-emerald-300 text-emerald-800 font-semibold py-2 px-4 rounded-md mb-4 w-full text-center">
                {branch.label}
              </div>
              <ul className="w-full space-y-2">
                {branch.sub.map((item, subIndex) => (
                  <li key={subIndex} className="bg-gray-50 text-gray-700 text-sm py-2 px-3 rounded border border-gray-200 text-center shadow-sm">
                    {item}
                  </li>
                ))}
              </ul>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};

const QuizContent = ({ initialQuestions, topic }) => {
  const [questions, setQuestions] = useState(initialQuestions);
  const [answers, setAnswers] = useState({});
  const [submitted, setSubmitted] = useState(false);
  const [isGenerating, setIsGenerating] = useState(false);

  // Reset when topic changes
  useEffect(() => {
    setQuestions(initialQuestions);
    setAnswers({});
    setSubmitted(false);
  }, [topic, initialQuestions]);

  const handleSelect = (qIndex, oIndex) => {
    if (submitted) return;
    setAnswers({ ...answers, [qIndex]: oIndex });
  };

  const calculateScore = () => {
    let score = 0;
    questions.forEach((q, i) => {
      if (answers[i] === q.correct) score++;
    });
    return score;
  };

  const generateDynamicQuestions = async () => {
    setIsGenerating(true);
    const systemInstruction = "You are a biology quiz generator. Generate 2 distinct multiple choice questions in Hebrew about the requested topic.";
    const prompt = `Topic: ${topic.title}\nContext text:\n${topic.theory.map(t => t.text).join('\n')}\n\nGenerate 2 new questions that are NOT in the standard quiz. Ensure they are educational.`;
    
    const schema = {
      type: "ARRAY",
      items: {
        type: "OBJECT",
        properties: {
          question: { type: "STRING" },
          options: { 
            type: "ARRAY", 
            items: { type: "STRING" }, 
            description: "Exactly 4 options in Hebrew" 
          },
          correct: { type: "INTEGER", description: "Index of correct option (0, 1, 2, or 3)" }
        },
        required: ["question", "options", "correct"]
      }
    };

    try {
      const newQuestions = await callGeminiAPI(prompt, systemInstruction, schema);
      if (newQuestions && newQuestions.length > 0) {
        setQuestions(prev => [...prev, ...newQuestions]);
        setSubmitted(false); // allow answering new questions
      }
    } catch (error) {
      alert("שגיאה ביצירת שאלות חדשות. נסה שוב.");
    } finally {
      setIsGenerating(false);
    }
  };

  return (
    <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
      {questions.map((q, qIndex) => (
        <div key={qIndex} className="mb-8 pb-6 border-b border-gray-100 last:border-0 animate-fade-in">
          <h4 className="text-lg font-bold mb-4 text-gray-800">{qIndex + 1}. {q.question}</h4>
          <div className="space-y-3">
            {q.options.map((opt, oIndex) => {
              let btnClass = "w-full text-right p-3 rounded-lg border transition-all ";
              if (submitted) {
                if (oIndex === q.correct) {
                  btnClass += "bg-green-100 border-green-500 text-green-800 font-bold";
                } else if (answers[qIndex] === oIndex) {
                  btnClass += "bg-red-100 border-red-500 text-red-800";
                } else {
                  btnClass += "bg-gray-50 border-gray-200 text-gray-500 opacity-50";
                }
              } else {
                if (answers[qIndex] === oIndex) {
                  btnClass += "bg-blue-100 border-blue-500 text-blue-800";
                } else {
                  btnClass += "bg-gray-50 border-gray-300 text-gray-700 hover:bg-gray-100 hover:border-blue-300";
                }
              }

              return (
                <button
                  key={oIndex}
                  onClick={() => handleSelect(qIndex, oIndex)}
                  disabled={submitted}
                  className={btnClass}
                >
                  {opt}
                </button>
              );
            })}
          </div>
        </div>
      ))}

      <div className="flex flex-col md:flex-row justify-between items-center gap-4 mt-8 border-t pt-6">
        {!submitted ? (
          <button 
            onClick={() => setSubmitted(true)}
            disabled={Object.keys(answers).length < questions.length}
            className="w-full md:w-auto px-8 py-3 bg-blue-600 text-white font-bold rounded-lg hover:bg-blue-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors"
          >
            בדוק תשובות
          </button>
        ) : (
          <div className="w-full md:w-auto flex-1 p-4 bg-blue-50 border-r-4 border-blue-500 rounded-lg flex items-center justify-between">
            <div className="text-xl font-bold text-blue-900">
              הציון שלך: {calculateScore()} / {questions.length}
            </div>
            <button 
              onClick={() => { setAnswers({}); setSubmitted(false); }}
              className="px-4 py-2 bg-white text-blue-600 border border-blue-200 rounded hover:bg-blue-50 font-medium"
            >
              נסה שוב
            </button>
          </div>
        )}

        {/* GEMINI FEATURE 2: Generate dynamic questions */}
        <button
          onClick={generateDynamicQuestions}
          disabled={isGenerating}
          className="w-full md:w-auto px-6 py-3 bg-gradient-to-r from-purple-500 to-indigo-600 text-white font-bold rounded-lg hover:from-purple-600 hover:to-indigo-700 transition-all shadow-md flex items-center justify-center gap-2 disabled:opacity-75"
        >
          {isGenerating ? <Loader2 className="w-5 h-5 animate-spin" /> : <Sparkles className="w-5 h-5 text-yellow-300" />}
          ✨ צור שאלות נוספות בעזרת AI
        </button>
      </div>
    </div>
  );
};


// --- MAIN APP ---

export default function App() {
  const [activeTab, setActiveTab] = useState('intro');
  const [activeSubSection, setActiveSubSection] = useState('theory'); // theory, map, quiz
  
  const tabs = Object.keys(learningData);
  const currentData = learningData[activeTab];

  return (
    <div dir="rtl" className="min-h-screen bg-gray-50 font-sans text-gray-800 pb-12 relative">
      <style>{`
        .perspective-1000 { perspective: 1000px; }
        .transform-style-3d { transform-style: preserve-3d; }
        .backface-hidden { backface-visibility: hidden; }
        .rotate-y-180 { transform: rotateY(180deg); }
        .animate-fade-in { animation: fadeIn 0.4s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
      `}</style>

      <header className="bg-gradient-to-r from-blue-700 to-indigo-800 text-white py-8 px-6 shadow-lg mb-8">
        <div className="max-w-4xl mx-auto">
          <h1 className="text-3xl font-extrabold mb-2 flex items-center gap-3">
            <Activity className="w-8 h-8" />
            יחידת למידה אינטראקטיבית: נגיפים
          </h1>
          <p className="text-blue-100 text-lg">למידה עצמאית לקראת בגרות בביולוגיה (מופעל ע"י בינה מלאכותית)</p>
        </div>
      </header>

      <main className="max-w-4xl mx-auto px-4 relative">
        <div className="flex flex-wrap gap-2 mb-8 bg-white p-2 rounded-xl shadow-sm border border-gray-200">
          {tabs.map((tab) => (
            <button
              key={tab}
              onClick={() => { setActiveTab(tab); setActiveSubSection('theory'); }}
              className={`flex-1 min-w-[120px] flex items-center justify-center gap-2 py-3 px-4 rounded-lg font-bold transition-all ${
                activeTab === tab 
                  ? 'bg-blue-100 text-blue-800 shadow-sm border border-blue-200' 
                  : 'text-gray-500 hover:bg-gray-50 hover:text-gray-700'
              }`}
            >
              {learningData[tab].icon}
              {learningData[tab].title}
            </button>
          ))}
        </div>

        <div className="bg-white rounded-2xl shadow-md border border-gray-100 overflow-hidden mb-8">
          <div className="flex border-b border-gray-200 bg-gray-50">
            <button 
              onClick={() => setActiveSubSection('theory')}
              className={`flex-1 py-4 font-semibold text-center border-b-2 transition-colors flex items-center justify-center gap-2 ${activeSubSection === 'theory' ? 'border-blue-600 text-blue-700 bg-white' : 'border-transparent text-gray-500 hover:text-gray-700'}`}
            >
              <BookOpen className="w-5 h-5 hidden sm:block" /> א. הקניית ידע
            </button>
            <button 
              onClick={() => setActiveSubSection('map')}
              className={`flex-1 py-4 font-semibold text-center border-b-2 transition-colors flex items-center justify-center gap-2 ${activeSubSection === 'map' ? 'border-blue-600 text-blue-700 bg-white' : 'border-transparent text-gray-500 hover:text-gray-700'}`}
            >
              <BrainCircuit className="w-5 h-5 hidden sm:block" /> ב. מפה וכרטיסיות
            </button>
            <button 
              onClick={() => setActiveSubSection('quiz')}
              className={`flex-1 py-4 font-semibold text-center border-b-2 transition-colors flex items-center justify-center gap-2 ${activeSubSection === 'quiz' ? 'border-blue-600 text-blue-700 bg-white' : 'border-transparent text-gray-500 hover:text-gray-700'}`}
            >
              <CheckSquare className="w-5 h-5 hidden sm:block" /> ג. תרגול אינטראקטיבי
            </button>
          </div>

          <div className="p-6 md:p-8 min-h-[400px]">
            {activeSubSection === 'theory' && (
              <div className="animate-fade-in">
                {currentData.poster && <PosterViewer poster={currentData.poster} />}
                {currentData.video && <VideoPlayer video={currentData.video} />}
                <div className="bg-amber-50 border-r-4 border-amber-400 p-4 mb-6 rounded-l-lg mt-8 shadow-sm">
                  <p className="text-amber-800 font-medium text-lg">
                    <strong>הנחיה לקריאה פעילה:</strong> קראו את הקטעים הבאים. לחצו על המילים המוסתרות (???) כדי לחשוף את מושגי המפתח.
                  </p>
                </div>
                <div className="space-y-4">
                  {currentData.theory.map((item, index) => (
                    <div key={index} className="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                      <InteractiveText text={item.text} />
                    </div>
                  ))}
                </div>
              </div>
            )}

            {activeSubSection === 'map' && (
              <div className="animate-fade-in">
                <h3 className="text-xl font-bold mb-6 text-gray-800 border-b pb-2">מפת חשיבה מסכמת</h3>
                <MindMap data={currentData.mindmap} />
                <h3 className="text-xl font-bold mt-12 mb-6 text-gray-800 border-b pb-2">כרטיסיות שינון (לחצו כדי להפוך)</h3>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                  {currentData.flashcards.map((card, index) => (
                    <Flashcard key={index} front={card.front} back={card.back} />
                  ))}
                </div>
              </div>
            )}

            {activeSubSection === 'quiz' && (
              <div className="animate-fade-in">
                <div className="bg-blue-50 border-r-4 border-blue-400 p-4 mb-6 rounded-l-lg flex justify-between items-center">
                  <p className="text-blue-800 font-medium">
                    בחן את עצמך! בחר את התשובה הנכונה ביותר. סיימת? לחץ על הכפתור למטה כדי ש-AI ייצר עבורך שאלות נוספות וחדשות! ✨
                  </p>
                </div>
                <QuizContent initialQuestions={currentData.quiz} topic={currentData} />
              </div>
            )}
          </div>
        </div>
      </main>

      {/* GEMINI FEATURE 1: AI Chat Assistant */}
      <AiChatBox currentTopic={currentData} />

    </div>
  );
}
