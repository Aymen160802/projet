# projet
import tkinter as tk
from tkinter import font, messagebox
import requests
import json

# Server configuration
SERVER_URL = "http://localhost:5000"  # Change to your server URL

# Arabic actions
actions = ["حجر", "ورق", "مقص"]

# Modern colors
BG_COLOR = "#2E3440"
BUTTON_COLOR = "#3B4252"
TEXT_COLOR = "#ECEFF4"
WIN_COLOR = "#A3BE8C"
LOSE_COLOR = "#BF616A"
TIE_COLOR = "#D08770"

class OnlineRockPaperScissors:
    def __init__(self, root):
        self.root = root
        self.root.title("لعبة الحجر-الورق-المقاس عبر الإنترنت")
        self.root.geometry("500x400")
        self.root.configure(bg=BG_COLOR)
        
        # Session ID
        self.session_id = None
        
        # Fonts
        self.custom_font = font.Font(family="Arial", size=12)
        self.title_font = font.Font(family="Arial", size=16, weight="bold")
        
        # Setup UI
        self.setup_ui()
        
        # Start game session
        self.start_game_session()
    
    def setup_ui(self):
        # Title
        title_label = tk.Label(
            self.root, 
            text="لعبة الحجر-الورق-المقاس (عبر الإنترنت)", 
            font=self.title_font, 
            bg=BG_COLOR, 
            fg=TEXT_COLOR
        )
        title_label.pack(pady=20)
        
        # Connection status
        self.connection_label = tk.Label(
            self.root,
            text="جاري الاتصال بالخادم...",
            font=self.custom_font,
            bg=BG_COLOR,
            fg=TEXT_COLOR
        )
        self.connection_label.pack()
        
        # Buttons frame
        button_frame = tk.Frame(self.root, bg=BG_COLOR)
        button_frame.pack(pady=20)
        
        # Action buttons
        self.buttons = []
        for i, action in enumerate(actions):
            btn = tk.Button(
                button_frame,
                text=action,
                font=self.custom_font,
                bg=BUTTON_COLOR,
                fg=TEXT_COLOR,
                width=10,
                height=2,
                command=lambda idx=i: self.play_round(idx),
                state=tk.DISABLED
            )
            btn.pack(side=tk.LEFT, padx=10)
            self.buttons.append(btn)
        
        # Result display
        self.result_label = tk.Label(
            self.root, 
            text="انتظر جاري الاتصال...", 
            font=self.custom_font, 
            bg=BG_COLOR, 
            fg=TEXT_COLOR
        )
        self.result_label.pack(pady=20)
        
        # Statistics frame
        stats_frame = tk.Frame(self.root, bg=BG_COLOR)
        stats_frame.pack()
        
        self.stats_labels = {
            "wins": tk.Label(stats_frame, text="انتصاراتك: 0", font=self.custom_font, bg=BG_COLOR, fg=WIN_COLOR),
            "losses": tk.Label(stats_frame, text="خسائرك: 0", font=self.custom_font, bg=BG_COLOR, fg=LOSE_COLOR),
            "ties": tk.Label(stats_frame, text="تعادلات: 0", font=self.custom_font, bg=BG_COLOR, fg=TIE_COLOR)
        }
        
        for label in self.stats_labels.values():
            label.pack(side=tk.LEFT, padx=10)
    
    def start_game_session(self):
        try:
            response = requests.post(f"{SERVER_URL}/start_game")
            data = response.json()
            self.session_id = data['session_id']
            self.connection_label.config(text="متصل بالخادم - جاهز للعب!")
            for btn in self.buttons:
                btn.config(state=tk.NORMAL)
            self.result_label.config(text="اختر أحد الخيارات أعلاه")
        except Exception as e:
            self.connection_label.config(text="فشل الاتصال بالخادم")
            messagebox.showerror("خطأ", "لا يمكن الاتصال بالخادم")
    
    def play_round(self, human_action):
        if not self.session_id:
            messagebox.showerror("خطأ", "لا يوجد جلسة لعبة نشطة")
            return
        
        try:
            response = requests.post(
                f"{SERVER_URL}/play",
                json={
                    'session_id': self.session_id,
                    'action': human_action
                }
            )
            data = response.json()
            
            # Update UI
            ai_action = data['ai_action']
            result = data['result']
            
            choice_text = f"اخترت: {actions[human_action]}  -  الذكاء الاصطناعي اختار: {actions[ai_action]}"
            
            if result == 0:
                result_text = "تعادل!"
                color = TIE_COLOR
            elif result == -1:
                result_text = "لقد فزت! 🎉"
                color = WIN_COLOR
            else:
                result_text = "فاز الذكاء الاصطناعي! 🤖"
                color = LOSE_COLOR
            
            self.result_label.config(text=f"{choice_text}\n{result_text}", fg=color)
            
            # Update stats
            stats = data['stats']
            self.stats_labels["wins"].config(text=f"انتصاراتك: {stats['wins']}")
            self.stats_labels["losses"].config(text=f"خسائرك: {stats['losses']}")
            self.stats_labels["ties"].config(text=f"تعادلات: {stats['ties']}")
            
        except Exception as e:
            messagebox.showerror("خطأ", "فشل في إرسال الحركة إلى الخادم")

if _name_ == "_main_":
    root = tk.Tk()
    try:
        game = OnlineRockPaperScissors(root)
        root.mainloop()
    except Exception as e:
        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")
localhos
