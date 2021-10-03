import argparse
import urllib
from urllib import request
import csv

	

class Server:
	def __init__(self, request_completion_time):
	    self.concurrent_request_time = request_completion_time
	    self.current_time = 0
	    self.current_request = None

	def start_next(self, new_request):
		self.current_request = new_request

	def busy(self):
	    if self.current_request != None:
	        return True
	    else:
	        return False
	

class Queue:
	def __init__(self):
		self.list = []

	def is_empty(self):
		if len(self.list) == 0:
			return True
		else:
			return False

	def enqueue(self, item):
		process = self.list.insert(0,item)

	def dequeue(self):
	    return self.list.pop()


	

class Request:
	def __init__(self, lst):
	    self.new_request_arrival_time = int(lst[0])
	    self.processing_time = int(lst[2])
	

	def wait_time(self, request_completion_time):
	    return request_completion_time - self.new_request_arrival_time


def simulateOneServer(row, request_queue, waiting_times):
	request = Request(row)
	request_queue.enqueue(request)
	request_completion_time = request.new_request_arrival_time
	concurrent_request_time = request_completion_time
	server = Server(request_completion_time)
	

	if (not server.busy()) and (not request_queue.is_empty()):
	    if sum(waiting_times) < request.new_request_arrival_time:
	        concurrent_request_time = request.new_request_arrival_time
	        request_completion_time = concurrent_request_time + request.processing_time
	        another_request = request_queue.dequeue()
	        waiting_times.append(another_request.wait_time(request_completion_time))
	        server.start_next(another_request)
	

	    else:
	        concurrent_request_time = request_completion_time
	        another_request = request_queue.dequeue()
	        request_completion_time = concurrent_request_time + request.processing_time
	        waiting_times.append(another_request.wait_time(request_completion_time))
	        server.start_next(another_request)
	else:
		print ("Server Is Busy or Request Is Empty")
	

	average_wait = sum(waiting_times) / len(waiting_times)
	print('Average Wait %6.2f secs %3d tasks remaining.' % (average_wait, len(request_queue.list)))

	return average_wait
	

def simulateManyServer(servers, row, request_queue, waiting_times):
	t_avg = 0
	for individual_server in servers:
		average_wait = simulateOneServer (row, request_queue, waiting_times)
		t_avg += average_wait
	return float(t_avg) / float(servers)
	

def main():
	parser = argparse.ArgumentParser()
	parser.add_argument("--file", help="Path To The Datafile", type=str, required=True)
	parser.add_argument("--servers", help="Number of Total Servers", type=str, required=False)
	args = parser.parse_args()

	file = args.file

	print("Processing Request for the file:", file)

	file_content = open(file, 'r').read()
	myreader = csv.reader(file_content.splitlines())

	request_queue = Queue()
	waiting_times = []
	total_average = 0
	c = 0

	servers = args.servers

	if servers:
		for row in myreader:
			avg = simulateManyServer(servers, row, request_queue, waiting_times)
			total_average += avg
			c += 1
		
		print ("Total Average", total_average/c)

	else:
		for row in myreader:
			avg = simulateOneServer(row, request_queue, waiting_times)
			total_average += avg
			c += 1

		print ("Total Average", total_average/c)


"""

Run One Server: python main.py --file requests.csv
Run Many Server: python main.py --file requests.csv --servers 2

"""
main ()
