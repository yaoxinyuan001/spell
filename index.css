import { useState, useEffect, useCallback, useMemo } from 'react';
import { motion, AnimatePresence } from 'motion/react';
import { 
  Volume2, 
  RotateCcw, 
  Trophy, 
  Timer, 
  Clock, 
  ChevronRight, 
  Undo2,
  Play,
  RefreshCw,
  CheckCircle2,
  XCircle,
  Sparkles,
  Loader2
} from 'lucide-react';
import { GoogleGenAI, Type } from "@google/genai";
import { Word } from './constants';

// Sound effects (using placeholder online resources)
const SOUNDS = {
  correct: 'https://assets.mixkit.co/active_storage/sfx/2013/2013-preview.mp3',
  wrong: 'https://assets.mixkit.co/active_storage/sfx/2018/2018-preview.mp3',
  click: 'https://assets.mixkit.co/active_storage/sfx/2568/2568-preview.mp3',
};

export default function App() {
  const [currentWord, setCurrentWord] = useState<Word | null>(null);
  const [userInput, setUserInput] = useState<string[]>([]);
  const [correctCount, setCorrectCount] = useState(0);
  const [timeLeft, setTimeLeft] = useState(30);
  const [totalTime, setTotalTime] = useState(0);
  const [gameState, setGameState] = useState<'start' | 'playing' | 'result'>('start');
  const [feedback, setFeedback] = useState<'correct' | 'wrong' | null>(null);
  const [shuffledLetters, setShuffledLetters] = useState<string[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  // AI Word Generation
  const fetchAIWord = async () => {
    setIsLoading(true);
    try {
      const apiKey = import.meta.env.VITE_GEMINI_API_KEY || process.env.GEMINI_API_KEY;
      
      if (!apiKey || apiKey === "undefined" || apiKey === "") {
        console.warn("Gemini API Key is missing or invalid. Using local fallback.");
        throw new Error("API_KEY_MISSING");
      }

      const ai = new GoogleGenAI({ apiKey });
      const response = await ai.models.generateContent({
        model: "gemini-flash-latest", // Using the most stable flash model
        contents: "Generate a random high-frequency IELTS vocabulary word (6-12 letters). Return ONLY a JSON object with keys: word, phonetic, translation, category. Example: {\"word\": \"challenge\", \"phonetic\": \"[ˈtʃælɪndʒ]\", \"translation\": \"挑战\", \"category\": \"Concept\"}",
        config: {
          responseMimeType: "application/json",
        }
      });

      if (!response.text) throw new Error("EMPTY_RESPONSE");

      // Clean the response text in case AI adds markdown blocks
      const cleanJson = response.text.replace(/```json|```/g, "").trim();
      const wordData = JSON.parse(cleanJson) as Word;
      
      return { 
        ...wordData, 
        word: wordData.word.toLowerCase().replace(/[^a-z]/g, "") // Ensure only letters
      };
    } catch (error) {
      console.error("AI Generation Error:", error);
      // Robust fallback word list
      const fallbacks = [
        { word: "academic", phonetic: "[ˌækəˈdemɪk]", translation: "学术的", category: "Quality" },
        { word: "strategy", phonetic: "[ˈstrætədʒi]", translation: "策略", category: "Concept" },
        { word: "efficient", phonetic: "[ɪˈfɪʃnt]", translation: "高效的", category: "Quality" },
        { word: "potential", phonetic: "[pəˈtenʃl]", translation: "潜力", category: "Concept" }
      ];
      return fallbacks[Math.floor(Math.random() * fallbacks.length)];
    } finally {
      setIsLoading(false);
    }
  };

  // Pronunciation
  const pronounce = useCallback((text: string) => {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'en-US';
    utterance.rate = 0.9;
    window.speechSynthesis.speak(utterance);
  }, []);

  const playSound = (type: keyof typeof SOUNDS) => {
    const audio = new Audio(SOUNDS[type]);
    audio.play().catch(() => {});
  };

  // Initialize word
  const initWord = useCallback(async () => {
    const wordObj = await fetchAIWord();
    setCurrentWord(wordObj);
    
    const letters = wordObj.word.split('');
    const pool = [...letters].sort(() => Math.random() - 0.5);
    setShuffledLetters(pool);
    setUserInput([]);
    setTimeLeft(30);
    setFeedback(null);
    
    setTimeout(() => pronounce(wordObj.word), 500);
  }, [pronounce]);

  // Start game
  const startGame = () => {
    setGameState('playing');
    setCorrectCount(0);
    setTotalTime(0);
    initWord();
  };

  // Timer logic
  useEffect(() => {
    let timer: number;
    if (gameState === 'playing' && !isLoading) {
      timer = window.setInterval(() => {
        setTimeLeft((prev) => {
          if (prev <= 1) {
            initWord();
            return 30;
          }
          return prev - 1;
        });
        setTotalTime((prev) => prev + 1);
      }, 1000);
    }
    return () => clearInterval(timer);
  }, [gameState, isLoading, initWord]);

  // Handle letter click
  const handleLetterClick = (letter: string, index: number) => {
    if (feedback || gameState !== 'playing' || !currentWord) return;
    
    playSound('click');
    const newInput = [...userInput, letter];
    setUserInput(newInput);

    const newPool = [...shuffledLetters];
    newPool.splice(index, 1);
    setShuffledLetters(newPool);

    if (newInput.length === currentWord.word.length) {
      const isCorrect = newInput.join('') === currentWord.word.toLowerCase();
      if (isCorrect) {
        setFeedback('correct');
        setCorrectCount((prev) => prev + 1);
        playSound('correct');
        pronounce(currentWord.word);
        setTimeout(() => initWord(), 1500);
      } else {
        setFeedback('wrong');
        playSound('wrong');
        setTimeout(() => {
          setFeedback(null);
          // Reset current attempt
          const letters = currentWord.word.split('');
          setShuffledLetters([...letters].sort(() => Math.random() - 0.5));
          setUserInput([]);
        }, 1000);
      }
    }
  };

  const handleUndo = () => {
    if (userInput.length === 0 || feedback) return;
    const lastLetter = userInput[userInput.length - 1];
    setUserInput(userInput.slice(0, -1));
    setShuffledLetters([...shuffledLetters, lastLetter].sort(() => Math.random() - 0.5));
  };

  const formatTime = (seconds: number) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  };

  return (
    <div className="min-h-screen bg-[#F5F5F0] text-[#1A1A1A] font-sans selection:bg-emerald-100">
      {/* Header */}
      <header className="fixed top-0 left-0 right-0 h-16 bg-white/80 backdrop-blur-md border-b border-black/5 flex items-center justify-between px-6 z-50">
        <div className="flex items-center gap-2">
          <div className="w-10 h-10 bg-emerald-500 rounded-xl flex items-center justify-center text-white shadow-lg shadow-emerald-200">
            <Trophy size={20} />
          </div>
          <div>
            <p className="text-[10px] uppercase tracking-wider font-bold text-black/40">Correct</p>
            <p className="text-lg font-bold leading-none">{correctCount}</p>
          </div>
        </div>

        <div className="flex items-center gap-8">
          <div className="flex flex-col items-center">
            <p className="text-[10px] uppercase tracking-wider font-bold text-black/40">Countdown</p>
            <div className={`flex items-center gap-1.5 font-mono text-xl font-bold ${timeLeft <= 5 ? 'text-red-500 animate-pulse' : 'text-emerald-600'}`}>
              <Timer size={18} />
              {timeLeft}s
            </div>
          </div>
          <div className="flex flex-col items-center">
            <p className="text-[10px] uppercase tracking-wider font-bold text-black/40">Total Time</p>
            <div className="flex items-center gap-1.5 font-mono text-xl font-bold text-black/60">
              <Clock size={18} />
              {formatTime(totalTime)}
            </div>
          </div>
        </div>

        <div className="flex items-center gap-4">
          <button 
            onClick={() => setGameState('result')}
            className="px-4 py-2 bg-black/5 hover:bg-black/10 rounded-xl text-xs font-bold transition-colors"
          >
            Finish Session
          </button>
          <button 
            onClick={() => setGameState('start')}
            className="p-2 hover:bg-black/5 rounded-full transition-colors"
          >
            <RotateCcw size={20} className="text-black/40" />
          </button>
        </div>
      </header>

      <main className="pt-24 pb-12 px-6 max-w-2xl mx-auto">
        <AnimatePresence mode="wait">
          {gameState === 'start' && (
            <motion.div 
              key="start"
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -20 }}
              className="flex flex-col items-center text-center py-20"
            >
              <div className="w-24 h-24 bg-emerald-100 rounded-3xl flex items-center justify-center text-emerald-600 mb-8 relative">
                <Play size={48} fill="currentColor" />
                <div className="absolute -top-2 -right-2 bg-white p-1.5 rounded-full shadow-sm border border-black/5">
                  <Sparkles size={16} className="text-amber-500" />
                </div>
              </div>
              <h1 className="text-4xl font-bold tracking-tight mb-4">AI IELTS Word Spelling</h1>
              <p className="text-black/50 mb-12 max-w-md">
                Powered by Gemini AI. Master dynamically generated IELTS vocabulary through an interactive spelling challenge.
              </p>
              <button 
                onClick={startGame}
                className="group relative px-12 py-4 bg-[#1A1A1A] text-white rounded-2xl font-bold text-lg overflow-hidden transition-transform active:scale-95"
              >
                <div className="absolute inset-0 bg-emerald-500 translate-y-full group-hover:translate-y-0 transition-transform duration-300" />
                <span className="relative z-10 flex items-center gap-2">
                  Launch AI Challenge <ChevronRight size={20} />
                </span>
              </button>
            </motion.div>
          )}

          {gameState === 'playing' && (
            <motion.div 
              key="playing"
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0 }}
              className="space-y-8"
            >
              {/* Word Card */}
              <div className="bg-white rounded-[32px] p-8 shadow-xl shadow-black/5 border border-black/5 relative overflow-hidden min-h-[400px] flex flex-col justify-center">
                {isLoading ? (
                  <div className="flex flex-col items-center justify-center space-y-4 py-20">
                    <Loader2 size={48} className="text-emerald-500 animate-spin" />
                    <p className="text-black/40 font-medium animate-pulse">Gemini is picking a word...</p>
                  </div>
                ) : currentWord && (
                  <>
                    {/* Illustration Placeholder */}
                    <div className="aspect-video w-full rounded-2xl bg-[#F9F9F7] mb-8 overflow-hidden relative group">
                      <img 
                        src={`https://picsum.photos/seed/${currentWord.word}/800/450`} 
                        alt={currentWord.word}
                        className="w-full h-full object-cover opacity-80 group-hover:scale-105 transition-transform duration-700"
                        referrerPolicy="no-referrer"
                      />
                      <div className="absolute inset-0 bg-gradient-to-t from-white/20 to-transparent" />
                      <div className="absolute top-4 left-4 flex gap-2">
                        <button 
                          onClick={() => initWord()}
                          disabled={isLoading}
                          className="px-3 py-1 bg-white/90 backdrop-blur rounded-full text-[10px] font-bold uppercase tracking-widest text-black/40 hover:text-emerald-600 transition-colors flex items-center gap-1"
                        >
                          <RefreshCw size={10} />
                          Skip
                        </button>
                      </div>
                      <div className="absolute top-4 right-4 px-3 py-1 bg-white/90 backdrop-blur rounded-full text-[10px] font-bold uppercase tracking-widest text-black/40 flex items-center gap-1">
                        <Sparkles size={10} className="text-amber-500" />
                        {currentWord.category}
                      </div>
                    </div>

                    <div className="text-center space-y-2">
                      <div className="flex items-center justify-center gap-3">
                        <span className="text-black/30 font-mono text-lg">{currentWord.phonetic}</span>
                        <button 
                          onClick={() => pronounce(currentWord.word)}
                          className="p-2 bg-emerald-50 text-emerald-600 rounded-full hover:bg-emerald-100 transition-colors"
                        >
                          <Volume2 size={18} />
                        </button>
                      </div>
                      <h2 className="text-3xl font-serif italic text-black/80">{currentWord.translation}</h2>
                    </div>
                  </>
                )}

                {/* Feedback Overlay */}
                <AnimatePresence>
                  {feedback && (
                    <motion.div 
                      initial={{ opacity: 0, scale: 0.8 }}
                      animate={{ opacity: 1, scale: 1 }}
                      exit={{ opacity: 0, scale: 0.8 }}
                      className="absolute inset-0 z-10 flex items-center justify-center backdrop-blur-sm bg-white/40"
                    >
                      {feedback === 'correct' ? (
                        <div className="flex flex-col items-center text-emerald-500">
                          <CheckCircle2 size={80} strokeWidth={1.5} />
                          <p className="text-xl font-bold mt-2">Excellent!</p>
                        </div>
                      ) : (
                        <div className="flex flex-col items-center text-red-500">
                          <XCircle size={80} strokeWidth={1.5} />
                          <p className="text-xl font-bold mt-2">Try Again</p>
                        </div>
                      )}
                    </motion.div>
                  )}
                </AnimatePresence>
              </div>

              {/* Blanks */}
              <div className="flex flex-wrap justify-center gap-2">
                {currentWord ? currentWord.word.split('').map((char, i) => (
                  <motion.div
                    key={i}
                    layoutId={`blank-${i}`}
                    className={`w-12 h-14 rounded-xl border-2 flex items-center justify-center text-2xl font-bold transition-all
                      ${userInput[i] 
                        ? 'bg-white border-emerald-500 text-emerald-600 shadow-lg shadow-emerald-100' 
                        : 'bg-black/5 border-transparent text-black/20'
                      }`}
                  >
                    {userInput[i] || ''}
                  </motion.div>
                )) : null}
              </div>

              {/* Letter Pool */}
              <div className="bg-white/50 backdrop-blur p-6 rounded-[24px] border border-black/5">
                <div className="flex flex-wrap justify-center gap-3">
                  {shuffledLetters.map((letter, i) => (
                    <motion.button
                      key={`${letter}-${i}`}
                      whileHover={{ y: -2 }}
                      whileTap={{ scale: 0.95 }}
                      onClick={() => handleLetterClick(letter, i)}
                      className="w-12 h-12 bg-white rounded-xl shadow-sm border border-black/5 flex items-center justify-center text-xl font-bold hover:shadow-md hover:border-emerald-200 transition-all"
                    >
                      {letter}
                    </motion.button>
                  ))}
                </div>
              </div>

              {/* Controls */}
              <div className="flex justify-center gap-4">
                <button 
                  onClick={handleUndo}
                  disabled={userInput.length === 0 || !!feedback || isLoading}
                  className="flex items-center gap-2 px-6 py-3 bg-white rounded-2xl border border-black/5 font-bold text-black/60 hover:bg-black/5 disabled:opacity-30 transition-all"
                >
                  <Undo2 size={18} /> Undo
                </button>
                <button 
                  onClick={() => {
                    if (currentWord) {
                      const letters = currentWord.word.split('');
                      setShuffledLetters([...letters].sort(() => Math.random() - 0.5));
                      setUserInput([]);
                    }
                  }}
                  disabled={isLoading}
                  className="flex items-center gap-2 px-6 py-3 bg-white rounded-2xl border border-black/5 font-bold text-black/60 hover:bg-black/5 disabled:opacity-30 transition-all"
                >
                  <RefreshCw size={18} /> Reset
                </button>
              </div>
            </motion.div>
          )}

          {gameState === 'result' && (
            <motion.div 
              key="result"
              initial={{ opacity: 0, scale: 0.9 }}
              animate={{ opacity: 1, scale: 1 }}
              className="text-center py-20"
            >
              <div className="w-24 h-24 bg-emerald-500 rounded-full flex items-center justify-center text-white mx-auto mb-8 shadow-2xl shadow-emerald-200">
                <Trophy size={48} />
              </div>
              <h2 className="text-4xl font-bold mb-2">Session Complete!</h2>
              <p className="text-black/40 mb-12">You've mastered some AI-generated IELTS words today.</p>
              
              <div className="grid grid-cols-2 gap-4 mb-12">
                <div className="bg-white p-6 rounded-3xl border border-black/5">
                  <p className="text-[10px] uppercase tracking-widest font-bold text-black/30 mb-1">Accuracy</p>
                  <p className="text-3xl font-bold text-emerald-600">{correctCount}</p>
                  <p className="text-xs text-black/40">Words Correct</p>
                </div>
                <div className="bg-white p-6 rounded-3xl border border-black/5">
                  <p className="text-[10px] uppercase tracking-widest font-bold text-black/30 mb-1">Duration</p>
                  <p className="text-3xl font-bold text-black/70">{formatTime(totalTime)}</p>
                  <p className="text-xs text-black/40">Total Time</p>
                </div>
              </div>

              <button 
                onClick={startGame}
                className="px-12 py-4 bg-[#1A1A1A] text-white rounded-2xl font-bold text-lg hover:scale-105 transition-transform active:scale-95"
              >
                Play Again
              </button>
            </motion.div>
          )}
        </AnimatePresence>
      </main>

      {/* Footer Decoration */}
      <footer className="fixed bottom-0 left-0 right-0 p-6 flex justify-center pointer-events-none">
        <p className="text-[10px] uppercase tracking-[0.2em] font-bold text-black/10">
          AI-Powered IELTS Mastery • Powered by Gemini
        </p>
      </footer>
    </div>
  );
}
