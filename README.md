import React, { useEffect, useMemo, useRef, useState } from "react";
import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Switch } from "@/components/ui/switch";
import { Plus, Trash2, Play, Pause, SkipForward, Volume2, Sparkles, NotebookText, Music4, Stars, Heart } from "lucide-react";

// ————————————————————————————————————————————————
// Celestial Hub – Totally Spies x Cosmic Girly vibe ✨
// Tabs: Music • Notes • Hype (Affirmations)
// - Replaces the Calendar tab with Notes + Hype
// - Saves playlist, notes, and custom affirmations to localStorage
// - Minimal, clean UI with rounded-2xl cards, soft shadows, and sparkly accents
// ————————————————————————————————————————————————

// Helpers for localStorage
const useLocalStorage = (key, initial) => {
  const [value, setValue] = useState(() => {
    try {
      const raw = localStorage.getItem(key);
      return raw ? JSON.parse(raw) : initial;
    } catch {
      return initial;
    }
  });
  useEffect(() => {
    try { localStorage.setItem(key, JSON.stringify(value)); } catch {}
  }, [key, value]);
  return [value, setValue];
};

const DEFAULT_AFFIRMATIONS = [
  "I am the main character—plot armor on.",
  "Soft life, hard boundaries.",
  "Money, opportunities, and ease are obsessed with me.",
  "I don’t chase—I attract. What’s mine simply arrives.",
  "My standards create my reality. Sprinkle, sprinkle.",
  "I am disciplined, divine, and deeply provided for.",
  "I glow different when I rest, hydrate, and mind my business.",
  "I am consistent. Consistency makes me unstoppable.",
];

const STAR_BG =
  "bg-[radial-gradient(ellipse_at_top,theme(colors.indigo.500/.25),transparent_60%),radial-gradient(ellipse_at_bottom,theme(colors.fuchsia.500/.25),transparent_60%)]";

export default function CelestialHubUpdated() {
  // Theme sparkle toggle (subtle animated stars)
  const [sparkle, setSparkle] = useLocalStorage("ch_sparkle", true);

  // ——— MUSIC STATE ———
  const [playlist, setPlaylist] = useLocalStorage("ch_playlist", []);
  const [title, setTitle] = useState("");
  const [url, setUrl] = useState("");
  const [file, setFile] = useState(null);
  const [currentIndex, setCurrentIndex] = useLocalStorage("ch_current_index", 0);
  const [isPlaying, setIsPlaying] = useState(false);
  const audioRef = useRef(null);
  const [volume, setVolume] = useLocalStorage("ch_volume", 0.85);

  // ——— NOTES STATE ———
  const [notes, setNotes] = useLocalStorage("ch_notes", "# Celestial Notes\n\n- Use this space for ideas, rituals, plans, or lyrics.\n- Tip: Press ⌘/Ctrl + B/I for **bold** or *italics* in your editor, or just keep it simple.");

  // ——— AFFIRMATIONS STATE ———
  const [customAffs, setCustomAffs] = useLocalStorage("ch_affs_custom", []);
  const pool = useMemo(() => (customAffs.length ? customAffs : DEFAULT_AFFIRMATIONS), [customAffs]);
  const [currentAff, setCurrentAff] = useLocalStorage("ch_aff_current", pool[0] || "You are luminous.");

  // Update audio element on volume or track
  useEffect(() => {
    if (audioRef.current) {
      audioRef.current.volume = volume;
    }
  }, [volume]);

  useEffect(() => {
    if (!audioRef.current) return;
    if (isPlaying) {
      audioRef.current.play().catch(() => setIsPlaying(false));
    } else {
      audioRef.current.pause();
    }
  }, [isPlaying, currentIndex]);

  // Handle track end → advance
  const onEnded = () => {
    if (currentIndex < playlist.length - 1) {
      setCurrentIndex(currentIndex + 1);
      setIsPlaying(true);
    } else {
      setIsPlaying(false);
    }
  };

  // Resolve current source (file or url)
  const currentSrc = useMemo(() => {
    if (!playlist.length) return "";
    const item = playlist[currentIndex];
    return item?.src || "";
  }, [playlist, currentIndex]);

  // Add track via URL or File
  const addTrack = () => {
    if (!title || (!url && !file)) return;
    let src = url.trim();
    let cleanup = null;
    if (file) {
      src = URL.createObjectURL(file);
      cleanup = src; // keep to revoke if deleted
    }
    const newItem = { id: crypto.randomUUID(), title: title.trim(), src, blobUrl: cleanup };
    setPlaylist([...playlist, newItem]);
    setTitle("");
    setUrl("");
    setFile(null);
  };

  const removeTrack = (id) => {
    const idx = playlist.findIndex((t) => t.id === id);
    if (idx === -1) return;
    const removed = playlist[idx];
    // Revoke blob URL if we created one
    if (removed.blobUrl) URL.revokeObjectURL(removed.blobUrl);

    const next = playlist.filter((t) => t.id !== id);
    setPlaylist(next);
    if (currentIndex >= next.length) setCurrentIndex(Math.max(0, next.length - 1));
  };

  const skipNext = () => {
    if (!playlist.length) return;
    setCurrentIndex((i) => (i + 1) % playlist.length);
    setIsPlaying(true);
  };

  const togglePlay = () => setIsPlaying((p) => !p);

  // Affirmation shuffle
  const shuffleAff = () => {
    if (!pool.length) return;
    const idx = Math.floor(Math.random() * pool.length);
    setCurrentAff(pool[idx]);
  };

  // Sparkle background animation
  const SparkleBackdrop = () => (
    <div className={`pointer-events-none absolute inset-0 -z-10 ${STAR_BG} ${sparkle ? "opacity-100" : "opacity-40"}`}>
      <div className="absolute inset-0 animate-pulse" />
    </div>
  );

  return (
    <div className={`min-h-screen w-full text-white ${STAR_BG} relative`}> 
      <SparkleBackdrop />

      {/* Header */}
      <header className="sticky top-0 z-20 backdrop-blur border-b border-white/10 bg-black/30">
        <div className="max-w-6xl mx-auto px-4 py-4 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <Stars className="h-6 w-6" />
            <h1 className="text-2xl font-semibold tracking-tight">Celestial Hub</h1>
            <span className="ml-2 text-xs px-2 py-1 rounded-full bg-fuchsia-500/20 border border-fuchsia-400/30 uppercase">v2 • Totally Spies Era</span>
          </div>
          <div className="flex items-center gap-3">
            <div className="flex items-center gap-2 text-sm">
              <span className="opacity-70">Sparkles</span>
              <Switch checked={sparkle} onCheckedChange={setSparkle} />
            </div>
          </div>
        </div>
      </header>

      {/* Body */}
      <main className="max-w-6xl mx-auto px-4 py-6">
        <Tabs defaultValue="music" className="w-full">
          <TabsList className="grid grid-cols-3 w-full bg-white/10">
            <TabsTrigger value="music" className="data-[state=active]:bg-white/20">
              <div className="flex items-center gap-2"><Music4 className="h-4 w-4"/> Music</div>
            </TabsTrigger>
            <TabsTrigger value="notes" className="data-[state=active]:bg-white/20">
              <div className="flex items-center gap-2"><NotebookText className="h-4 w-4"/> Notes</div>
            </TabsTrigger>
            <TabsTrigger value="hype" className="data-[state=active]:bg-white/20">
              <div className="flex items-center gap-2"><Sparkles className="h-4 w-4"/> Hype</div>
            </TabsTrigger>
          </TabsList>

          {/* MUSIC TAB */}
          <TabsContent value="music" className="mt-6">
            <div className="grid lg:grid-cols-3 gap-6">
              {/* Player */}
              <Card className="bg-black/40 border-white/10 col-span-2">
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Music4 className="h-5 w-5"/> Cosmic Player</CardTitle>
                </CardHeader>
                <CardContent className="space-y-4">
                  <div className="rounded-2xl border border-white/10 p-4 bg-white/5">
                    <div className="text-sm opacity-80">Now Playing</div>
                    <div className="text-lg font-semibold truncate">{playlist[currentIndex]?.title || "—"}</div>
                    <audio ref={audioRef} src={currentSrc} onEnded={onEnded} className="w-full mt-3" controls />

                    <div className="mt-3 flex items-center gap-3">
                      <Button variant="secondary" onClick={togglePlay} className="rounded-2xl">
                        {isPlaying ? <Pause className="h-4 w-4"/> : <Play className="h-4 w-4"/>}
                        <span className="ml-2">{isPlaying ? "Pause" : "Play"}</span>
                      </Button>
                      <Button variant="secondary" onClick={skipNext} className="rounded-2xl"><SkipForward className="h-4 w-4"/><span className="ml-2">Next</span></Button>
                      <div className="ml-auto flex items-center gap-2 text-sm opacity-80"><Volume2 className="h-4 w-4"/>Vol</div>
                      <input type="range" min={0} max={1} step={0.01} value={volume} onChange={(e)=>setVolume(parseFloat(e.target.value))} className="accent-fuchsia-400"/>
                    </div>
                  </div>

                  {/* Add track */}
                  <div className="grid md:grid-cols-3 gap-3">
                    <Input placeholder="Title (e.g., Energy Cleanse Mix)" value={title} onChange={(e)=>setTitle(e.target.value)} />
                    <Input placeholder="Paste audio URL (mp3/m4a/stream)" value={url} onChange={(e)=>setUrl(e.target.value)} />
                    <div className="flex gap-2">
                      <Input type="file" accept="audio/*" onChange={(e)=>setFile(e.target.files?.[0]||null)} />
                      <Button onClick={addTrack} className="whitespace-nowrap rounded-2xl"><Plus className="h-4 w-4"/><span className="ml-2">Add</span></Button>
                    </div>
                  </div>
                </CardContent>
              </Card>

              {/* Playlist */}
              <Card className="bg-black/40 border-white/10">
                <CardHeader>
                  <CardTitle>Playlist</CardTitle>
                </CardHeader>
                <CardContent className="space-y-2">
                  {playlist.length === 0 && (
                    <div className="text-sm opacity-70">
                      No tracks yet. Add a URL or upload a file above. Pro tip: save a YouTube-to-MP3 link you own rights to, or host audio on a trusted CDN.
                    </div>
                  )}
                  <ul className="space-y-2">
                    {playlist.map((t, i) => (
                      <li key={t.id} className={`flex items-center gap-3 rounded-xl border border-white/10 px-3 py-2 ${i===currentIndex? "bg-white/10" : "bg-white/5"}`}>
                        <button onClick={()=>{setCurrentIndex(i); setIsPlaying(true);}} className="text-left flex-1 truncate">
                          <div className="text-sm font-medium truncate">{t.title}</div>
                          <div className="text-xs opacity-60 truncate">{t.src}</div>
                        </button>
                        <Button size="icon" variant="ghost" onClick={()=>removeTrack(t.id)} className="hover:bg-fuchsia-500/20">
                          <Trash2 className="h-4 w-4"/>
                        </Button>
                      </li>
                    ))}
                  </ul>
                </CardContent>
              </Card>
            </div>
          </TabsContent>

          {/* NOTES TAB */}
          <TabsContent value="notes" className="mt-6">
            <Card className="bg-black/40 border-white/10">
              <CardHeader>
                <CardTitle className="flex items-center gap-2"><NotebookText className="h-5 w-5"/> Cosmic Notes</CardTitle>
              </CardHeader>
              <CardContent className="grid lg:grid-cols-2 gap-6">
                <div className="space-y-3">
                  <Textarea value={notes} onChange={(e)=>setNotes(e.target.value)} className="min-h-[360px] bg-white/5 border-white/10"/>
                  <div className="text-xs opacity-70">
                    Saved locally. Markdown-friendly (basic). Copy/paste into your README or journal when ready.
                  </div>
                </div>
                <div className="rounded-2xl border border-white/10 bg-white/5 p-4 prose prose-invert max-w-none">
                  <h3 className="mb-2">Preview</h3>
                  <MarkdownLite text={notes} />
                </div>
              </CardContent>
            </Card>
          </TabsContent>

          {/* HYPE / AFFIRMATIONS TAB */}
          <TabsContent value="hype" className="mt-6">
            <div className="grid lg:grid-cols-3 gap-6">
              <Card className="bg-black/40 border-white/10 col-span-2">
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Sparkles className="h-5 w-5"/> Daily Hype</CardTitle>
                </CardHeader>
                <CardContent className="space-y-4">
                  <div className="rounded-3xl p-6 border border-fuchsia-400/30 bg-gradient-to-br from-fuchsia-600/30 to-indigo-600/30 shadow-lg">
                    <p className="text-2xl leading-snug font-semibold flex items-start gap-3"><Heart className="h-6 w-6 shrink-0 mt-1"/>{currentAff}</p>
                    <div className="mt-4 flex gap-3">
                      <Button className="rounded-2xl" onClick={shuffleAff}><Sparkles className="h-4 w-4"/><span className="ml-2">New Hype</span></Button>
                      <Button variant="secondary" className="rounded-2xl" onClick={()=>navigator.clipboard.writeText(currentAff)}>Copy</Button>
                    </div>
                  </div>

                  <div className="text-sm opacity-80">
                    Tip: Add your own mantras below. Your customs override the default pool when present.
                  </div>
                </CardContent>
              </Card>

              <Card className="bg-black/40 border-white/10">
                <CardHeader>
                  <CardTitle>Custom Mantras</CardTitle>
                </CardHeader>
                <CardContent>
                  <AffList value={customAffs} onChange={setCustomAffs} />
                </CardContent>
              </Card>
            </div>
          </TabsContent>
        </Tabs>
      </main>

      {/* Footer */}
      <footer className="py-8 text-center text-xs opacity-70">
        Built with love • Cosmic + Girly • "Totally Spies" energy ✨
      </footer>
    </div>
  );
}

// ——— Tiny Markdown previewer (safe-lite) ———
function MarkdownLite({ text }) {
  const html = useMemo(() => {
    try {
      // extra-light markdown: **bold**, *italics*, #/## headings, - bullets, \n -> <br>
      let t = (text || "").replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
      t = t.replace(/^###\s(.+)$/gm, '<h3>$1<\/h3>');
      t = t.replace(/^##\s(.+)$/gm, '<h2>$1<\/h2>');
      t = t.replace(/^#\s(.+)$/gm, '<h1>$1<\/h1>');
      t = t.replace(/\*\*(.+?)\*\*/g, '<strong>$1<\/strong>');
      t = t.replace(/\*(.+?)\*/g, '<em>$1<\/em>');
      // bullets
      t = t.replace(/^\-\s(.+)$/gm, '<li>$1<\/li>');
      t = t.replace(/(<li>.*<\/li>)/gs, '<ul>$1<\/ul>');
      t = t.replace(/\n/g, '<br/>');
      return t;
    } catch {
      return text;
    }
  }, [text]);
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// ——— Editable list for custom affirmations ———
function AffList({ value = [], onChange }) {
  const [draft, setDraft] = useState("");
  const add = () => {
    const v = draft.trim();
    if (!v) return;
    onChange([...(value || []), v]);
    setDraft("");
  };
  const del = (idx) => {
    const next = [...value];
    next.splice(idx, 1);
    onChange(next);
  };
  return (
    <div className="space-y-3">
      <div className="flex gap-2">
        <Input placeholder="Type a mantra (e.g., ‘I am a magnet for aligned opportunities.’)" value={draft} onChange={(e)=>setDraft(e.target.value)} />
        <Button onClick={add} className="rounded-2xl"><Plus className="h-4 w-4"/></Button>
      </div>
      {(!value || value.length===0) && (
        <div className="text-sm opacity-70">No custom mantras yet.</div>
      )}
      <ul className="space-y-2">
        {value?.map((v, i) => (
          <li key={i} className="flex items-center gap-2 rounded-xl border border-white/10 bg-white/5 px-3 py-2">
            <span className="flex-1 text-sm">{v}</span>
            <Button size="icon" variant="ghost" onClick={()=>del(i)} className="hover:bg-fuchsia-500/20"><Trash2 className="h-4 w-4"/></Button>
          </li>
        ))}
      </ul>
      {value?.length ? (
        <div className="text-xs opacity-70">Saved locally. These replace defaults when present.</div>
      ) : null}
    </div>
  );
}
