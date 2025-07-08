import time, json, os
from datetime import datetime
import matplotlib.pyplot as plt

class StudyTracker:
    def __init__(self):
        self.start_time = None
        self.data_file = "study_data.json"
        self.data = self.load_data()

    def load_data(self):
        if os.path.exists(self.data_file):
            with open(self.data_file, 'r') as f:
                return json.load(f)
        return {}

    def save_data(self):
        with open(self.data_file, 'w') as f:
            json.dump(self.data, f, indent=4)

    def get_today(self):
        return datetime.now().strftime("%Y-%m-%d")

    def start(self):
        self.start_time = time.time()

    def stop(self, subject):
        if not self.start_time:
            return 0
        elapsed = time.time() - self.start_time
        self.start_time = None
        date = self.get_today()
        self.data.setdefault(date, {})
        self.data[date][subject] = self.data[date].get(subject, 0) + elapsed
        self.save_data()
        return int(elapsed // 60)

    def add_manual(self, subject, minutes):
        seconds = int(minutes) * 60
        date = self.get_today()
        self.data.setdefault(date, {})
        self.data[date][subject] = self.data[date].get(subject, 0) + seconds
        self.save_data()

    def get_today_data(self):
        return self.data.get(self.get_today(), {})

    def show_today_summary(self):
        data = self.get_today_data()
        if not data:
            print("오늘 학습 데이터가 없습니다.")
            return
        for subject, seconds in data.items():
            print(f"- {subject}: {int(seconds // 60)}분")

    def get_advice(self):
        data = self.get_today_data()
        if not data:
            print("오늘 학습 기록이 없습니다.")
            return
        total = sum(data.values())
        most = max(data, key=data.get)
        least = min(data, key=data.get)
        print(f"📊 오늘 총 학습 시간: {int(total // 60)}분")
        print(f"- 가장 많이 공부한 과목: {most}")
        print(f"- 가장 적게 공부한 과목: {least}")
        print(f"📚 '{least}' 과목을 조금 더 보충해보세요!")

    def plot_graph(self):
        data = self.get_today_data()
        if not data:
            print("📉 그래프를 그릴 데이터가 없습니다.")
            return

        subjects = list(data.keys())
        minutes = [int(t // 60) for t in data.values()]

        plt.figure(figsize=(6, 4))
        bars = plt.bar(subjects, minutes, color='mediumseagreen')
        plt.title("오늘의 과목별 학습 시간")
        plt.ylabel("시간 (분)")
        for bar, m in zip(bars, minutes):
            plt.text(bar.get_x() + bar.get_width()/2, bar.get_height()+1, f"{m}분", ha='center')
        plt.tight_layout()
        plt.savefig("study_graph.png")
        plt.close()
        print("📈 그래프가 'study_graph.png' 파일로 저장되었습니다.")

# 콘솔 메뉴
def main():
    tracker = StudyTracker()

    while True:
        print("\n📚 쉬운 학습 트래커")
        print("1. 공부 시작")
        print("2. 공부 종료")
        print("3. 수동 시간 추가")
        print("4. 오늘 학습 요약")
        print("5. 학습 조언 받기")
        print("6. 그래프 저장")
        print("0. 종료")
        choice = input("선택: ").strip()

        if choice == "1":
            subject = input("과목 이름: ")
            tracker.start()
            print(f"⏱️ {subject} 공부 시작!")

        elif choice == "2":
            subject = input("과목 이름: ")
            mins = tracker.stop(subject)
            print(f"✅ {subject} 공부 종료 ({mins}분)")

        elif choice == "3":
            subject = input("과목 이름: ")
            minutes = input("추가할 시간 (분): ")
            if minutes.isdigit():
                tracker.add_manual(subject, minutes)
                print(f"➕ {subject}에 {minutes}분 추가됨.")
            else:
                print("숫자를 정확히 입력하세요.")

        elif choice == "4":
            tracker.show_today_summary()

        elif choice == "5":
            tracker.get_advice()

        elif choice == "6":
            tracker.plot_graph()

        elif choice == "0":
            print("👋 종료합니다.")
            break
        else:
            print("❗ 올바른 번호를 선택하세요.")

if __name__ == "__main__":
    main()
