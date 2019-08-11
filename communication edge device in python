import re
import threading
import time
from Queue import Queue

import keyboard

import BlochsCode.SecurityVisionData
import Communication.ServerConnection
import Vision_Processing.CarVisionData

if __name__ == "__main__":
    print("Server running")

    msg_pattern = re.compile("(^\w*MSG)")
    useless_number_pattern = re.compile("(\d+$)")

    # Booleans that decide if we process the images
    process_car_vision = True
    process_security_vision = True
    get_comp2_processed_data = True

    pi_connection = Communication.ServerConnection.ServerConnection(
        Communication.ServerConnection.LISTENER,
        Communication.ServerConnection.PI_PORT)

    data_from_comp2 = Queue()


    def get_comp2_data():
        print "comp2 starting"
        conn = Communication.ServerConnection.ServerConnection(
            Communication.ServerConnection.LISTENER,
            Communication.ServerConnection.COMP2_PORT)
        while True:

            try:
                msg = conn.get_msg()
                if not msg:
                    continue

                # print("Recieved msg: " + msg)

                messages = msg.split('MESSAGE')
                for real_msg in messages:
                    if real_msg == '':
                        continue

                    stripped = real_msg.strip()
                    removed_useless_num = useless_number_pattern.sub(
                        '', stripped)

                    msg_as_was_sent = "MESSAGE" + removed_useless_num
                    # print("Recieved msg from comp 2: " + msg_as_was_sent)
                    data_from_comp2.put(msg_as_was_sent)
                    time.sleep(0.1)

            except Exception as e:
                print("EXCEPTION CAUGHT")
                print(str(e))

if get_comp2_processed_data:
    periodic_loop_thread_comp2 = threading.Thread(
        target=get_comp2_data, args=())
    periodic_loop_thread_comp2.start()
    print "comp2 server up"

data_from_car = Queue()


def get_vision_data():
    car_vision_data = Vision_Processing.CarVisionData.CarVisionData(
        pi_connection)
    while True:
        car_bloons = car_vision_data.get_bloons()

        can_shoot = car_vision_data.get_can_shoot()
        did_pop = car_vision_data.get_did_pop()

        data_from_car.put("MESSAGECarBloonsMSG" + str(car_bloons))
        data_from_car.put("MESSAGECanShootMSG" + str(can_shoot))
        data_from_car.put("MESSAGEDidPopMSG" + str(did_pop))
        time.sleep(0.1)


if process_car_vision:
    periodic_loop_thread_car_vision = threading.Thread(
        target=get_vision_data, args=())
    periodic_loop_thread_car_vision.start()
    print "car vision server is up"

data_from_vision = Queue()


def process_security_vision_data():
    security_vision_data = BlochsCode.SecurityVisionData.SecurityVisionData()
    while True:
        room_bloons = security_vision_data.get_bloons()

        if room_bloons:
            print("Room Bloons:")
            print(room_bloons)

        data_from_vision.put("MESSAGERoomBloons1MSG" + str(room_bloons))

        time.sleep(0.1)


if process_security_vision:
    periodic_loop_thread_comp2 = threading.Thread(
        target=process_security_vision_data, args=())
    periodic_loop_thread_comp2.start()
    print "security vision thread up"


def update_pressed_hotkey():
    """
    Function that is called by keyboard to update the flag if the hotkey
    was pressed
    """
    pressed_hotkey = True


pressed_hotkey = False  # flag if hotkey of ctrl+enter was pressed
keyboard.add_hotkey('ctrl+enter', update_pressed_hotkey)
# Starts tracking if hotkey was pressed

safety_stopped = False

while True:
    # sending messages
    if pressed_hotkey:
        print("Changing safety stop to " + str(not safety_stopped))

        pressed_hotkey = False
        safety_stopped = not safety_stopped

    pi_connection.send_msg("MESSAGECarWorkingMSG" + str(safety_stopped))
    # print "sending..."


    try:
        if not data_from_vision.empty():
            while not data_from_vision.empty():
                msg = data_from_vision.get()
                pi_connection.send_msg(msg)

        if not data_from_comp2.empty():
            while not data_from_comp2.empty():
                msg = data_from_comp2.get()
                pi_connection.send_msg(msg)

        if not data_from_car.empty():
            while not data_from_car.empty():
                msg = data_from_car.get()
                pi_connection.send_msg(msg)

    except Exception as e:
        print("Exception in server main loop")
        print(e)

    time.sleep(0.2)
