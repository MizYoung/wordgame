# wordgame
word chain in Korean 
def korean_word_game():
    print("=== 한국어 끝말잇기 게임을 시작합니다! ===")
    print("종료하려면 '종료'를 입력하세요.\n")
    
    used_words = set()
    current_word = ""

    # 컴퓨터가 첫 단어를 제시 (실제 개발 시에는 단어 데이터베이스에서 무작위 추출)
    current_word = "기차"
    used_words.add(current_word)
    print(f"컴퓨터 제시어: {current_word}")

    while True:
        # 사용자 입력
        user_word = input(f"'{current_word[-1]}'로 시작하는 단어를 입력하세요: ").strip()

        # 게임 종료 조건
        if user_word == "종료":
            print("게임을 종료합니다.")
            break

        # 규칙 1: 글자 수 검사 (최소 2글자 이상)
        if len(user_word) < 2:
            print("❌ 두 글자 이상의 단어만 입력 가능합니다. 다시 시도하세요.")
            continue

        # 규칙 2: 첫 글자 일치 여부 검사
        if user_word[0] != current_word[-1]:
            print(f"❌ 틀렸습니다! '{current_word[-1]}'로 시작해야 합니다.")
            continue

        # 규칙 3: 중복 단어 검사
        if user_word in used_words:
            print(f"❌ 이미 사용된 단어입니다: '{user_word}'. 다시 시도하세요.")
            continue

        # 모든 조건을 통과한 경우
        used_words.add(user_word)
        print(f"▶️ 인정! 입력한 단어: {user_word}")
        
        # 임시 컴퓨터 턴 (실제로는 단어 DB에서 user_word[-1]로 시작하는 단어를 찾아야 합니다)
        # 예시를 위해 사용자가 입력한 단어의 마지막 글자로 시작하는 임의의 단어를 지정합니다.
        last_char = user_word[-1]
        
        if last_char == "음":
            computer_word = "음악"
        elif last_char == "악":
            computer_word = "악기"
        elif last_char == "기":
            computer_word = "기상"
        elif last_char == "상":
            computer_word = "상자"
        else:
            print(f"\n🎉 컴퓨터가 '{last_char}'로 시작하는 단어를 찾지 못했습니다! 당신의 승리!")
            break

        if computer_word in used_words:
            print(f"\n🎉 컴퓨터가 중복 단어({computer_word})를 말했습니다! 당신의 승리!")
            break

        current_word = computer_word
        used_words.add(current_word)
        print(f"\n🖥️ 컴퓨터: {current_word}")

# 게임 실행
if __name__ == "__main__":
    korean_word_game()
