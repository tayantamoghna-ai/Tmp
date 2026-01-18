import React, { useState, useRef } from 'react';
import { Upload, Image as ImageIcon, Type, Sparkles, X, Download, Wand2, RefreshCw, Shirt, Layers, AlertCircle } from 'lucide-react';

// --- API Configuration ---
const apiKey = ""; // System will inject the key at runtime

// --- Helper: Resize and Convert Image to Base64 ---
// This prevents "Payload too large" errors by ensuring images aren't massive
const processImage = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = (event) => {
      const img = new Image();
      img.src = event.target.result;
      img.onload = () => {
        const canvas = document.createElement('canvas');
        let width = img.width;
        let height = img.height;
        const maxDim = 1024; // Limit max dimension to 1024px for API stability

        if (width > maxDim || height > maxDim) {
          if (width > height) {
            height *= maxDim / width;
            width = maxDim;
          } else {
            width *= maxDim / height;
            height = maxDim;
          }
        }

        canvas.width = width;
        canvas.height = height;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0, width, height);

        // Get Base64
        const dataUrl = canvas.toDataURL(file.type === 'image/png' ? 'image/png' : 'image/jpeg', 0.9);
        const base64 = dataUrl.split(',')[1];
        const mimeType = dataUrl.split(';')[0].split(':')[1];
        
        resolve({ base64, mimeType, preview: dataUrl });
      };
      img.onerror = (e) => reject(e);
    };
    reader.onerror = (e) => reject(e);
  });
};

// --- Components ---

const Button = ({ children, onClick, variant = 'primary', disabled, className = '', icon: Icon }) => {
  const baseStyle = "flex items-center justify-center gap-2 px-6 py-3 rounded-xl font-semibold transition-all duration-200 disabled:opacity-50 disabled:cursor-not-allowed transform active:scale-95";
  const variants = {
    primary: "bg-gradient-to-r from-violet-600 to-indigo-600 hover:from-violet-500 hover:to-indigo-500 text-white shadow-lg shadow-indigo-500/20",
    secondary: "bg-gray-800 hover:bg-gray-700 text-gray-200 border border-gray-700",
    outline: "border-2 border-violet-500/50 text-violet-400 hover:bg-violet-500/10",
  };

  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`${baseStyle} ${variants[variant]} ${className}`}
    >
      {Icon && <Icon size={20} />}
      {children}
    </button>
  );
};

const ImageUploader = ({ image, onUpload, onClear, label, icon: Icon, id }) => {
  const inputRef = useRef(null);
  const [dragActive, setDragActive] = useState(false);

  const handleDrag = (e) => {
    e.preventDefault();
    e.stopPropagation();
    if (e.type === "dragenter" || e.type === "dragover") {
      setDragActive(true);
    } else if (e.type === "dragleave") {
      setDragActive(false);
    }
  };

  const handleDrop = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setDragActive(false);
    if (e.dataTransfer.files && e.dataTransfer.files[0]) {
      onUpload(e.dataTransfer.files[0]);
    }
  };

  const handleChange = (e) => {
    if (e.target.files && e.target.files[0]) {
      onUpload(e.target.files[0]);
    }
  };

  return (
    <div 
      className={`relative group flex flex-col items-center justify-center w-full h-64 border-2 border-dashed rounded-2xl transition-all duration-300 overflow-hidden ${
        dragActive 
          ? "border-violet-500 bg-violet-500/10 scale-[1.02]" 
          : image 
            ? "border-violet-500/30 bg-gray-900" 
            : "border-gray-700 bg-gray-800/50 hover:border-gray-600 hover:bg-gray-800"
      }`}
      onDragEnter={handleDrag}
      onDragLeave={handleDrag}
      onDragOver={handleDrag}
      onDrop={handleDrop}
    >
      {image ? (
        <>
          <img 
            src={image.preview} 
            alt="Upload" 
            className="w-full h-full object-contain p-2"
          />
          <div className="absolute inset-0 bg-black/60 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center gap-4 backdrop-blur-sm">
            <button 
              onClick={() => inputRef.current.click()}
              className="p-3 bg-white/10 hover:bg-white/20 rounded-full text-white transition-colors"
              title="Replace Image"
            >
              <RefreshCw size={24} />
            </button>
            <button 
              onClick={onClear}
              className="p-3 bg-red-500/20 hover:bg-red-500/40 text-red-400 rounded-full transition-colors"
              title="Remove Image"
            >
              <X size={24} />
            </button>
          </div>
        </>
      ) : (
        <div 
          className="flex flex-col items-center justify-center cursor-pointer w-full h-full"
          onClick={() => inputRef.current.click()}
        >
          <div className="p-4 bg-gray-800 rounded-full mb-3 group-hover:scale-110 transition-transform duration-300 shadow-xl">
            <Icon size={32} className="text-violet-400" />
          </div>
          <p className="text-gray-300 font-medium mb-1">{label}</p>
          <p className="text-gray-500 text-sm">Click or drag & drop image</p>
        </div>
      )}
      <input
        ref={inputRef}
        type="file"
        id={id}
        className="hidden"
        accept="image/*"
        onChange={handleChange}
      />
    </div>
  );
};

const ModeCard = ({ active, onClick, icon: Icon, title, description }) => (
  <div 
    onClick={onClick}
    className={`cursor-pointer p-4 rounded-xl border transition-all duration-200 flex flex-col gap-2 ${
      active 
        ? 'bg-violet-600/10 border-violet-500 shadow-[0_0_20px_rgba(139,92,246,0.15)]' 
        : 'bg-gray-800 border-gray-700 hover:border-gray-600 hover:bg-gray-750'
    }`}
  >
    <div className={`p-2 w-fit rounded-lg ${active ? 'bg-violet-500 text-white' : 'bg-gray-700 text-gray-400'}`}>
      <Icon size={20} />
    </div>
    <div>
      <h3 className={`font-semibold ${active ? 'text-violet-300' : 'text-gray-200'}`}>{title}</h3>
      <p className="text-xs text-gray-500 leading-relaxed mt-1">{description}</p>
    </div>
  </div>
);

// --- Main App Component ---

export default function App() {
  const [sourceImage, setSourceImage] = useState(null);
  const [refImage, setRefImage] = useState(null);
  const [resultImage, setResultImage] = useState(null);
  const [prompt, setPrompt] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const [mode, setMode] = useState('text'); // 'text', 'reference', 'hybrid'

  const handleSourceUpload = async (file) => {
    try {
      const processed = await processImage(file);
      setSourceImage({ file, ...processed });
      setError("");
    } catch (e) {
      console.error(e);
      setError("Failed to process source image. Try a smaller file.");
    }
  };

  const handleRefUpload = async (file) => {
    try {
      const processed = await processImage(file);
      setRefImage({ file, ...processed });
      setError("");
    } catch (e) {
      console.error(e);
      setError("Failed to process reference image. Try a smaller file.");
    }
  };

  const generateImage = async () => {
    if (!sourceImage) {
      setError("Please upload a photo of a person first.");
      return;
    }
    if (mode === 'text' && !prompt.trim()) {
      setError("Please describe the new outfit.");
      return;
    }
    if (mode === 'reference' && !refImage) {
      setError("Please upload a reference image for the clothing.");
      return;
    }
    if (mode === 'hybrid' && (!refImage || !prompt.trim())) {
      setError("Please upload a reference image and provide a description.");
      return;
    }

    setLoading(true);
    setError("");
    setResultImage(null);

    try {
      // 1. Construct the specific prompt based on mode
      let finalPrompt = "";
      const parts = [];

      // Stronger preservation instruction
      const preservationInstruction = "Strictly preserve the person's face, identity, hair, pose, and background. ONLY change the clothing.";

      if (mode === 'text') {
        finalPrompt = `Edit this image. Change the person's outfit to: ${prompt}. ${preservationInstruction}`;
        
        parts.push({ text: finalPrompt });
        parts.push({
          inlineData: {
            mimeType: sourceImage.mimeType,
            data: sourceImage.base64
          }
        });
      } 
      else if (mode === 'reference') {
        finalPrompt = `Edit the first image. Replace the person's clothes with the clothing shown in the second image. ${preservationInstruction}`;
        
        parts.push({ text: finalPrompt });
        parts.push({
          inlineData: {
            mimeType: sourceImage.mimeType,
            data: sourceImage.base64
          }
        });
        parts.push({
          inlineData: {
            mimeType: refImage.mimeType,
            data: refImage.base64
          }
        });
      } 
      else if (mode === 'hybrid') {
        finalPrompt = `Edit the first image. Using the second image as a style reference, change the person's clothing. Additional instruction: ${prompt}. ${preservationInstruction}`;
        
        parts.push({ text: finalPrompt });
        parts.push({
          inlineData: {
            mimeType: sourceImage.mimeType,
            data: sourceImage.base64
          }
        });
        parts.push({
          inlineData: {
            mimeType: refImage.mimeType,
            data: refImage.base64
          }
        });
      }

      // 2. API Call with Retries and Better Error Handling
      const makeApiCall = async (attempt = 1) => {
        try {
          const response = await fetch(
            `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`,
            {
              method: 'POST',
              headers: {
                'Content-Type': 'application/json',
              },
              body: JSON.stringify({
                contents: [{ parts: parts }],
                generationConfig: {
                  responseModalities: ["TEXT", "IMAGE"],
                  temperature: 0.4, 
                }
              })
            }
          );

          // SAFE PARSING LOGIC:
          // 1. Check if response is ok. If not, get text error.
          // 2. If response is ok, get text, then parse JSON.
          
          if (!response.ok) {
            const errorText = await response.text();
            let errorMsg = `API Error ${response.status}`;
            try {
              const errorJson = JSON.parse(errorText);
              if (errorJson.error && errorJson.error.message) {
                errorMsg += `: ${errorJson.error.message}`;
              }
            } catch (e) {
               // If error text isn't JSON, use a substring
               errorMsg += `: ${errorText.substring(0, 100)}`;
            }
            throw new Error(errorMsg);
          }

          const responseText = await response.text();
          if (!responseText) throw new Error("Received empty response from server.");
          
          const result = JSON.parse(responseText);

          // Extract image from response
          const outputData = result.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData;
          
          if (outputData) {
            return `data:${outputData.mimeType};base64,${outputData.data}`;
          } else {
             const textRefusal = result.candidates?.[0]?.content?.parts?.find(p => p.text)?.text;
             // Sometimes the model finishes because of safety without refusal text
             const finishReason = result.candidates?.[0]?.finishReason;

             if (textRefusal) throw new Error("Model Message: " + textRefusal);
             if (finishReason === 'SAFETY') throw new Error("The image generation was blocked by safety settings.");
             
             throw new Error("No image generated. Please try a different prompt or photo.");
          }

        } catch (err) {
          // Retry on server errors (5xx) or network issues, but not 4xx (client errors) or Safety blocks
          const isRetryable = err.message.includes("500") || err.message.includes("503") || err.message.includes("fetch");
          
          if (attempt < 4 && isRetryable) {
            await new Promise(r => setTimeout(r, 1000 * Math.pow(2, attempt))); // Exponential backoff
            return makeApiCall(attempt + 1);
          }
          throw err;
        }
      };

      const generatedImageBase64 = await makeApiCall();
      setResultImage(generatedImageBase64);

    } catch (err) {
      console.error(err);
      setError(err.message || "An unexpected error occurred while generating.");
    } finally {
      setLoading(false);
    }
  };

  const handleDownload = () => {
    if (!resultImage) return;
    const link = document.createElement("a");
    link.href = resultImage;
    link.download = `tmp-ai-generated-${Date.now()}.png`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };

  return (
    <div className="min-h-screen bg-[#0a0a0c] text-white selection:bg-violet-500/30 font-sans">
      
      {/* Header */}
      <header className="border-b border-gray-800 bg-[#0a0a0c]/80 backdrop-blur-md sticky top-0 z-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="w-8 h-8 rounded-lg bg-gradient-to-tr from-violet-600 to-indigo-600 flex items-center justify-center shadow-lg shadow-violet-500/20">
              <Shirt className="text-white w-5 h-5" />
            </div>
            <span className="text-xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-white to-gray-400">
              TMP ai
            </span>
          </div>
          <div className="text-xs font-mono text-gray-600 hidden sm:block">
            Powered by Gemini Nano Banana
          </div>
        </div>
      </header>

      <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8 lg:py-12">
        <div className="grid lg:grid-cols-12 gap-8 lg:gap-12">
          
          {/* Left Column: Inputs */}
          <div className="lg:col-span-5 space-y-8">
            
            {/* Step 1: Source Image */}
            <section className="space-y-4">
              <div className="flex items-center justify-between">
                <h2 className="text-lg font-semibold text-gray-200 flex items-center gap-2">
                  <span className="flex items-center justify-center w-6 h-6 rounded-full bg-gray-800 text-xs text-gray-400 border border-gray-700">1</span>
                  Upload Person
                </h2>
                {sourceImage && (
                  <span className="text-xs text-green-400 font-medium bg-green-400/10 px-2 py-1 rounded-full border border-green-400/20">
                    Ready
                  </span>
                )}
              </div>
              <ImageUploader 
                image={sourceImage}
                onUpload={handleSourceUpload}
                onClear={() => setSourceImage(null)}
                label="Upload photo of person"
                icon={Upload}
                id="source-upload"
              />
            </section>

            {/* Step 2: Mode Selection */}
            <section className="space-y-4">
              <h2 className="text-lg font-semibold text-gray-200 flex items-center gap-2">
                <span className="flex items-center justify-center w-6 h-6 rounded-full bg-gray-800 text-xs text-gray-400 border border-gray-700">2</span>
                Select Method
              </h2>
              <div className="grid grid-cols-1 sm:grid-cols-3 gap-3">
                <ModeCard 
                  active={mode === 'text'} 
                  onClick={() => setMode('text')} 
                  icon={Type} 
                  title="Text" 
                  description="Describe the new outfit" 
                />
                <ModeCard 
                  active={mode === 'reference'} 
                  onClick={() => setMode('reference')} 
                  icon={ImageIcon} 
                  title="Image" 
                  description="Upload dress reference" 
                />
                <ModeCard 
                  active={mode === 'hybrid'} 
                  onClick={() => setMode('hybrid')} 
                  icon={Layers} 
                  title="Hybrid" 
                  description="Image + Text tweaks" 
                />
              </div>
            </section>

            {/* Step 3: Specific Inputs */}
            <section className="space-y-4 animate-in fade-in slide-in-from-top-4 duration-300">
              <h2 className="text-lg font-semibold text-gray-200 flex items-center gap-2">
                <span className="flex items-center justify-center w-6 h-6 rounded-full bg-gray-800 text-xs text-gray-400 border border-gray-700">3</span>
                Customize
              </h2>
              
              <div className="p-5 bg-gray-800/50 rounded-2xl border border-gray-700/50 space-y-4">
                {(mode === 'text' || mode === 'hybrid') && (
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-2">
                      {mode === 'hybrid' ? 'Modification Instructions' : 'Describe the outfit'}
                    </label>
                    <textarea
                      value={prompt}
                      onChange={(e) => setPrompt(e.target.value)}
                      placeholder={mode === 'hybrid' ? "e.g., Make it red instead of blue, add long sleeves..." : "e.g., A vintage leather jacket over a white t-shirt and blue jeans..."}
                      className="w-full h-32 bg-gray-900 border border-gray-700 rounded-xl p-4 text-gray-200 placeholder-gray-500 focus:ring-2 focus:ring-violet-500 focus:border-transparent outline-none resize-none transition-all"
                    />
                  </div>
                )}

                {(mode === 'reference' || mode === 'hybrid') && (
                  <div>
                     <label className="block text-sm font-medium text-gray-400 mb-2">
                      Reference Clothing Image
                    </label>
                    <div className="h-48">
                      <ImageUploader 
                        image={refImage}
                        onUpload={handleRefUpload}
                        onClear={() => setRefImage(null)}
                        label="Upload outfit reference"
                        icon={Shirt}
                        id="ref-upload"
                      />
                    </div>
                  </div>
                )}
              </div>
            </section>

            {/* Generate Button */}
            <div className="pt-2 pb-8 lg:pb-0">
              <Button 
                onClick={generateImage} 
                disabled={loading} 
                className="w-full text-lg shadow-violet-900/20"
                icon={Sparkles}
              >
                {loading ? 'Processing Magic...' : 'Generate New Look'}
              </Button>
              {error && (
                <div className="mt-4 p-4 bg-red-500/10 border border-red-500/20 rounded-xl text-red-400 text-sm flex items-start gap-2">
                  <AlertCircle className="w-5 h-5 shrink-0 mt-0.5" />
                  <span className="break-all">{error}</span>
                </div>
              )}
            </div>
          </div>

          {/* Right Column: Result */}
          <div className="lg:col-span-7 flex flex-col h-full min-h-[500px]">
            <div className="flex-1 bg-gray-900/50 border border-gray-800 rounded-3xl relative overflow-hidden flex flex-col items-center justify-center p-4 lg:p-8">
              
              {/* Grid Background Pattern */}
              <div className="absolute inset-0 opacity-20 pointer-events-none" 
                   style={{ 
                     backgroundImage: 'linear-gradient(#2d2d30 1px, transparent 1px), linear-gradient(90deg, #2d2d30 1px, transparent 1px)', 
                     backgroundSize: '20px 20px' 
                   }}>
              </div>

              {resultImage ? (
                <div className="relative z-10 w-full h-full flex flex-col items-center justify-center animate-in zoom-in-95 duration-500">
                  <div className="relative rounded-xl overflow-hidden shadow-2xl shadow-black/50 max-h-[70vh]">
                    <img 
                      src={resultImage} 
                      alt="Generated" 
                      className="max-w-full max-h-full object-contain"
                    />
                  </div>
                  <div className="mt-6 flex gap-4">
                    <Button 
                      onClick={handleDownload} 
                      variant="secondary"
                      icon={Download}
                    >
                      Download
                    </Button>
                    <Button 
                      onClick={() => setResultImage(null)} 
                      variant="outline"
                      icon={Wand2}
                    >
                      Try Another
                    </Button>
                  </div>
                </div>
              ) : (
                <div className="text-center z-10 max-w-sm px-6">
                  {loading ? (
                    <div className="flex flex-col items-center gap-4">
                      <div className="relative">
                        <div className="w-16 h-16 rounded-full border-4 border-violet-500/30 border-t-violet-500 animate-spin"></div>
                        <div className="absolute inset-0 flex items-center justify-center">
                          <Sparkles size={20} className="text-violet-400 animate-pulse" />
                        </div>
                      </div>
                      <h3 className="text-xl font-semibold text-white">Designing Outfit...</h3>
                      <p className="text-gray-400">The AI is tailoring the clothes. This might take a few seconds.</p>
                    </div>
                  ) : (
                    <div className="flex flex-col items-center gap-4 opacity-50">
                      <div className="w-24 h-24 rounded-full bg-gray-800 flex items-center justify-center mb-2">
                        <ImageIcon size={40} className="text-gray-600" />
                      </div>
                      <h3 className="text-xl font-semibold text-gray-300">Ready to Transform</h3>
                      <p className="text-gray-500">
                        Upload a photo and choose your method on the left to see the magic happen here.
                      </p>
                    </div>
                  )}
                </div>
              )}
            </div>
            
            {/* Info Footer */}
            <div className="mt-4 text-center lg:text-right">
              <p className="text-xs text-gray-600">
                Tip: For best results, use a source photo with good lighting and a clear view of the body.
              </p>
            </div>
          </div>

        </div>
      </main>
    </div>
  );
}
