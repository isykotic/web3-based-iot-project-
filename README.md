# web3-based-iot-project-
sensor data storing in eth blockchain with a parallel functionality of Realtime viewing and limiting access by putting access specifiers in place 

import time
import ssl
import json
from web3 import Web3, HTTPProvider
from Adafruit_BME280 import *
from Adafruit_DHT import *
from WiFiClientSecure import *
from WebSocketsClient import *

# replace with your WiFi credentials
ssid = "your_SSID"
password = "your_PASSWORD"

# replace with your server's information
host = "yourhost.com"
port = 443  # port 443 is for HTTPS
fingerprint = "cert"  # replace with the fingerprint of the SSL/TLS certificate

# initialize sensors
bme = BME280(t_mode=BME280_OSAMPLE_8, p_mode=BME280_OSAMPLE_8, h_mode=BME280_OSAMPLE_8)
dht_pin = 2
dht_type = DHT22
dht = DHT(dht_pin, dht_type)
tds_pin = 0
ppm_conversion_factor = 0.5
mq9_pin = 1

# initialize Ethereum node and account information
web3 = Web3(HTTPProvider("http://localhost:8545"))  # replace with your Ethereum node's IP address and port number
private_key = "your private key"  # replace with your Ethereum account's private key
contract_address = "0x..."  # replace with your smart contract's address
abi = json.loads('[{"inputs":[{"internalType":"uint256","name":"_temperature","type":"uint256"},{"internalType":"uint256","name":"_humidity","type":"uint256"},{"internalType":"uint256","name":"_pressure","type":"uint256"},{"internalType":"uint256","name":"_altitude","type":"uint256"},{"internalType":"uint256","name":"_tds","type":"uint256"},{"internalType":"uint256","name":"_co2","type":"uint256"},{"internalType":"uint256","name":"_timestamp","type":"uint256"}],"name":"addReading","outputs":[],"stateMutability":"nonpayable","type":"function"}]')  # replace with your smart contract's ABI

# initialize WebSocket client
client = WiFiClientSecure()
client.setFingerprint(fingerprint)
web_socket = WebSocketsClient()

# initialize prime number generator
MAX_N = 1000000
primes = [True] * MAX_N
prime_numbers = []


def connect_wifi():
    print("Connecting to WiFi...")
    attempts = 0
    while not client.connect(host, port) and attempts < 10:
        time.sleep(1)
        attempts += 1
    if not client.connected():
        print("Failed to connect to WiFi")
        while True:
            pass
    print("WiFi connected!")
    print("IP address: ", client.localIP())


def read_sensors():
    bme_result = bme.read_temperature_humidity_pressure()
    if not bme_result:
        print("Could not find a valid BME280 sensor, check wiring!")
        return None
    dht_result = dht.read_retry(dht_type, dht_pin)
    if dht_result:
        temperature, humidity = dht_result
    else:
        temperature, humidity = None, None
    tds_result = analogRead(tds_pin)
    tds = (tds_result / 1024.0) * 5.0 * ppm_conversion_factor * 1000.0
    co2_result = analogRead(mq9_pin)
    co2 = ...  # add code to convert the analog reading to a concentration value

    # validate the sensor readings
    if temperature is not None and humidity is not None
    # validate the sensor readings
    if temperature is not None and humidity is not None and tds_result is not None and co2_result is not None:
        return {"temperature": int(temperature * 100), "humidity": int(humidity * 100), "pressure": int(bme_result.pressure), "altitude": int(bme_result.altitude), "tds": int(tds), "co2": int(co2), "timestamp": int(time.time())}
    else:
        return None


def send_reading():
    reading = read_sensors()
    if reading is not None:
        print("Sending reading:", reading)
        # sign the transaction
        nonce = web3.eth.getTransactionCount(web3.eth.accounts[0])
        transaction = {"from": web3.eth.accounts[0], "to": contract_address, "nonce": nonce, "gasPrice": web3.toWei("20", "gwei"), "gas": 100000, "data": web3.toHex(text=json.dumps({"function": "addReading", "args": [reading["temperature"], reading["humidity"], reading["pressure"], reading["altitude"], reading["tds"], reading["co2"], reading["timestamp"]]}))}
        signed_transaction = web3.eth.account.signTransaction(transaction, private_key)
        # send the transaction
        tx_hash = web3.eth.sendRawTransaction(signed_transaction.rawTransaction)
        print("Transaction sent with hash:", web3.toHex(tx_hash))


def on_message(ws, message):
    print("Received message:", message)


def on_error(ws, error):
    print("Error:", error)


def on_close(ws):
    print("WebSocket closed")


def on_open(ws):
    print("WebSocket opened")
    ws.send("hello world")


def generate_prime_numbers():
    for i in range(2, MAX_N):
        if primes[i]:
            prime_numbers.append(i)
            for j in range(i * i, MAX_N, i):
                primes[j] = False


if __name__ == "__main__":
    connect_wifi()
    web_socket.set_ssl(True)
    web_socket.set_root_ca("cert")
    web_socket.connect("wss://yourhost.com:443")
    web_socket.on_message = on_message
    web_socket.on_error = on_error
    web_socket.on_close = on_close
    web_socket.on_open = on_open
    while True:
        send_reading()
        web_socket.run_forever()
