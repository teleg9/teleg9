import os
import sys
import time
import threading
from datetime import datetime

from Data.config import root_path

from Data.data import file_List, streamer_list, site_streamer_list
from Data.config import Discord_message, restart_time
from Data.discord_send import Discord_start, Discord_end, Discord_Error

from Recording.online.AF_online import africa
from Recording.online.YouTube_online import youtube
from Recording.online.Kick_online import kick
from Recording.online.Twitch_online import twitch
from Recording.online.Twitcas_online import twitcasting
from Recording.online.Panda_online import panda
from Recording.online.chzzk_online import chzzk

from Recording.AF import africa_recoding
from Recording.YouTube import youtube_recoding
from Recording.Kick import kick_recoding
from Recording.Twitch import twitch_recoding
from Recording.Twitcas import twitcasting_recoding
from Recording.Panda import panda_recoding
from Recording.chzzk import chzzk_recoding

First = True # 최초실행
Zero_base = 'Zero_base'

#--------------------------------------------------------------------------
#0 = 플랫폼 1번의 이름(메인)
#1 = 플랫폼 1번의 온라인
#2 = 플랫폼 1번의 녹화상태
#3 = 플랫폼 1번의 비밀번호
#4 = 플랫폼 1번의 마지막 방송 종료시간

#5 = 플랫폼 2번의 이름(동일 닉네임 다른 사이트 존재)
#6 = 플랫폼 2번의 온라인
#7 = 플랫폼 2번의 녹화상태
#8 = 플랫폼 2번의 비밀번호
#9 = 플랫폼 2번의 마지막 방송 종료시간

#10 = 방제
#11 = 별명
#--------------------------------------------------------------------------

#녹화 플랫폼 추가되면 변경 할 부분
def target(filename):
    if filename == '아프리카':
        target_form = 'africa'

    elif filename == '유튜브':
        target_form = 'youtube'

    elif filename == '킥':
        target_form = 'kick'

    elif filename == '트위치':
        target_form = 'twitch'

    elif filename == '트윗캐스트':
        target_form = 'twitcasting'

    elif filename == '판다':
        target_form = 'panda'

    elif filename == '치지직':
        target_form = 'chzzk'

    else:
        target_form = None
                
    if not os.path.exists(root_path):
        os.mkdir(root_path)
        
    if not os.path.exists(os.path.join(root_path, target_form)):
        os.mkdir(os.path.join(root_path, target_form))

    return target_form
#------------------------------------

def read_List(): #리스트 추출
    List_directory = './List'
    new_filenames = []
    streamer_name = {}

    if os.path.exists(List_directory): #녹화 리스트 파일 추가
        for dirpath, dirnames, filenames in os.walk(List_directory):
            for filename in filenames:
                if not filename.startswith('기간녹화'):
                    file_name = filename.split(".")[0].strip()
                    new_filenames.append(file_name)

    if First != True: #녹화 리스트 파일 최신화
        for _file_List in file_List:
            if not _file_List in new_filenames:
                file_List.remove(_file_List)

    if new_filenames != []:
        for filename in new_filenames:
            filename_path = os.path.join('./List', filename + '.txt')
            with open(filename_path, "r", encoding="utf-8", errors='ignore') as file:
                lines = file.read().splitlines()

            for line in lines:
                if line.strip():
                    password = Zero_base
                    info = line.split("/")
                    streamer = line.split("/")[0].strip() #녹화대상
                    Nick_name = line.split("/")[1].strip() #설명
                    target_form = target(filename) #플랫폼
                    
                    if len(info) > 2: #비밀번호
                        password = line.split("/")[2].strip()
                        password = password.strip()

                    streamer = streamer.strip() #공백제거
                    Nick_name = Nick_name.strip()
                    target_form = target_form.strip()
                    streamer_name[(streamer, target_form)] = target_form

                    if First == True: #최초 실행
                        if streamer in list(streamer_list.keys()) and streamer_list[streamer][0] != target_form: #딕셔너리에 있고 다른 플랫폼일때
                            streamer_list[streamer][5] = target_form
                            streamer_list[streamer][8] = password
                            
                        else: #딕셔너리에 없을때
                            streamer_list[streamer] = [
                                target_form, 'offline', 'record_False', password, Zero_base,
                                Zero_base, 'offline', 'record_False', Zero_base, Zero_base,
                                Zero_base, Nick_name]

                    if First == False: #최신화
                        if not streamer in list(streamer_list.keys()): #딕셔너리에 없을때
                            streamer_list[streamer] = [
                                target_form, 'offline', 'record_False', password, Zero_base,
                                Zero_base, 'offline', 'record_False', Zero_base, Zero_base,
                                Zero_base, Nick_name]

                        if streamer in list(streamer_list.keys()) and streamer_list[streamer][0] != target_form: #딕셔너리에 있고 다른 플랫폼일때
                            streamer_list[streamer][5] = target_form
                            streamer_list[streamer][8] = password

                        if streamer in list(streamer_list.keys()) and password != Zero_base:
                            if streamer_list[streamer][0] == target_form and streamer_list[streamer][3] != password:
                                streamer_list[streamer][3] = password
                                
                            if streamer_list[streamer][5] == target_form and streamer_list[streamer][8] != password:
                                streamer_list[streamer][8] = password

        for _streamer_list in list(streamer_list.keys()): #제거 로직
            if streamer_list[_streamer_list][5] == Zero_base: #다른 플랫폼이 없을때
                if not (_streamer_list, streamer_list[_streamer_list][0]) in streamer_name.keys(): #(스트리머, 플랫폼)딕셔너리에 없을때
                    del streamer_list[_streamer_list]
                    
            else: #다른 플랫폼이 있을때
                
                #1번째와 2번째 (스트리머, 플랫폼)딕셔너리에 없을때
                if (_streamer_list, streamer_list[_streamer_list][0]) not in streamer_name.keys() and (_streamer_list, streamer_list[_streamer_list][5]) not in streamer_name.keys():
                    del streamer_list[_streamer_list]

                #1번째 (스트리머, 플랫폼)딕셔너리에 없을때
                elif (_streamer_list, streamer_list[_streamer_list][0]) not in streamer_name.keys():
                    streamer_list[_streamer_list][0] = streamer_list[_streamer_list][5]
                    streamer_list[_streamer_list][5] = Zero_base
                    streamer_list[_streamer_list][1] = streamer_list[_streamer_list][6]
                    streamer_list[_streamer_list][6] = 'offline'
                    streamer_list[_streamer_list][2] = streamer_list[_streamer_list][7]
                    streamer_list[_streamer_list][7] = 'record_False'
                    streamer_list[_streamer_list][3] = streamer_list[_streamer_list][8]
                    streamer_list[_streamer_list][8] = Zero_base
                    streamer_list[_streamer_list][4] = streamer_list[_streamer_list][9]
                    streamer_list[_streamer_list][9] = Zero_base

                #2번째 (스트리머, 플랫폼)딕셔너리에 없을때
                elif (_streamer_list, streamer_list[_streamer_list][5]) not in streamer_name.keys():
                    streamer_list[_streamer_list][5] = Zero_base
                    streamer_list[_streamer_list][6] = 'offline'
                    streamer_list[_streamer_list][7] = 'record_False'
                    streamer_list[_streamer_list][8] = Zero_base
                    streamer_list[_streamer_list][9] = Zero_base

def multy_online_check(): #온라인 파악
    try:
        threads = []
        if site_streamer_list != {}: #온라인 체크 인원 초기화
            for site in list(site_streamer_list.keys()):
                del site_streamer_list[site]

        for _streamer_list in list(streamer_list.keys()):
            if streamer_list[_streamer_list][0] != Zero_base and streamer_list[_streamer_list][2] == 'record_False':
                if streamer_list[_streamer_list][0] in list(site_streamer_list.keys()):
                    if not _streamer_list in list(site_streamer_list[streamer_list[_streamer_list][0]]):
                        site_streamer_list[streamer_list[_streamer_list][0]].append(_streamer_list)
                else:
                    site_streamer_list[streamer_list[_streamer_list][0]] = [f'{_streamer_list}']
                    
            if streamer_list[_streamer_list][5] != Zero_base and streamer_list[_streamer_list][7] == 'record_False':
                if streamer_list[_streamer_list][5] in list(site_streamer_list.keys()):
                    if not _streamer_list in list(site_streamer_list[streamer_list[_streamer_list][5]]):
                        site_streamer_list[streamer_list[_streamer_list][5]].append(_streamer_list)
                else:
                    site_streamer_list[streamer_list[_streamer_list][5]] = [f'{_streamer_list}']

        if len(site_streamer_list.keys()) != 0:
            for site in list(site_streamer_list.keys()):
                online_target = globals().get(site)
                thread = threading.Thread(target=online_target)
                threads.append(thread)
                thread.start()

                for thread in threads:
                    thread.join()
                    
    except Exception as e:
        print(e)
        time.sleep(1)

def multy_recording(): #녹화 시작
    try:
        threads = []
        for _streamer_list in list(streamer_list.keys()):            
            with open('./Data/Discord.token', 'r', encoding='utf8') as f:
                wornning = f.read()
                
            if streamer_list[_streamer_list][1] == 'online' and streamer_list[_streamer_list][2] == 'record_False': #녹화 상태 변경 및 녹화 함수 타겟팅
                streamer_list[_streamer_list][2] = 'record_True'
                online_target = globals().get(streamer_list[_streamer_list][0] + '_recoding')
                messege = f'{streamer_list[_streamer_list][11]} 녹화 시작 - {streamer_list[_streamer_list][0]}'
                if Discord_message and wornning == 'True':
                    Discord_start(messege)

                print(f'{streamer_list[_streamer_list][11]} 녹화 시작')
                thread = threading.Thread(target=online_target, args = (_streamer_list,))
                threads.append(thread)
                thread.start()
                
            if streamer_list[_streamer_list][6] == 'online' and streamer_list[_streamer_list][7] == 'record_False':
                streamer_list[_streamer_list][7] = 'record_True'
                online_target = globals().get(streamer_list[_streamer_list][5] + '_recoding')
                messege = f'{streamer_list[_streamer_list][11]} 녹화 시작 - {streamer_list[_streamer_list][5]}'
                if Discord_message and wornning == 'True':
                    Discord_start(messege)

                print(f'{streamer_list[_streamer_list][11]} 녹화 시작')
                thread = threading.Thread(target=online_target, args = (_streamer_list,))
                threads.append(thread)
                thread.start()
                    
    except Exception as e:
        print(e)
        time.sleep(1)

if __name__ == "__main__":
    read_List() #최초실행
    First = False
    
    while True:
        try:
            start_time = time.time()
            online_count = 0
            online_list = {}
            read_List() #리스트 최신화
            multy_online_check()
            multy_recording()
            
            for _streamer_list in list(streamer_list.keys()): #녹화인원 파악
                if streamer_list[_streamer_list][1] == 'online' and streamer_list[_streamer_list][2] != 'record_False':
                    if streamer_list[_streamer_list][0] in list(online_list.keys()):
                        online_list[streamer_list[_streamer_list][0]].append(streamer_list[_streamer_list][11])
                    else:
                        online_list[streamer_list[_streamer_list][0]] = [f'{streamer_list[_streamer_list][11]}']
                    online_count += 1
                    
                if streamer_list[_streamer_list][6] == 'online' and streamer_list[_streamer_list][7] != 'record_False':
                    if streamer_list[_streamer_list][5] in list(online_list.keys()):
                        online_list[streamer_list[_streamer_list][5]].append(streamer_list[_streamer_list][11])
                    else:
                        online_list[streamer_list[_streamer_list][5]] = [f'{streamer_list[_streamer_list][11]}']
                    online_count += 1

            if online_list != {}:
                for _online_list in list(online_list.keys()):
                    print(f'{_online_list} - {online_list[_online_list]}')
                
            print(f'{online_count}/{len(streamer_list.keys())}이 녹화 중') # 총인원
            
            end_time = time.time() - start_time
            if end_time >= 15:
                continue
            
            time.sleep(15 - end_time)
            
        except KeyboardInterrupt:
            print(f"실녹기가 사용자에 의해 중지되었습니다.\n잠시 기다려주십시오.")
            break

        except OSError as e:
            print(f"OSError: {e}")
            time.sleep(1)
            
        except Exception as e:
            print(e)
            print(f"1초 후에 다시 시도합니다.")
            time.sleep(1)

