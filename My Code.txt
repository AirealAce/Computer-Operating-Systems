# header: includes classes, functions, libraries, etc..
######################################################################################################################################################################
class process:
    def __init__(self,name, bursts):
        self.name = name        #process name
        self.bursts = bursts    #all the bursts
        self.cpu = []           #cpu bursts
        self.io = []            #io bursts
        for i in range(len(bursts)):
            if i%2 == 0:        #get CPU bursts
                self.cpu.append(bursts[i])
            elif i%2 == 1:      #get IO bursts
                self.io.append(bursts[i])
        self.rdy = 0                #ready time: when the process is able to cpu burst
        self.job = self.cpu[0]      #job time length: the length of the next cpu burst to be executed
        self.pr = 1                 #priority
        self.tw = 0                 #process wait time: time spent in ready queue due to other process hogging cpu with their burst
        self.ttr = 0                #process turnaround time: time process fully executes - response time
        self.tr = -1                #process response time: the start for the first cpu burst
    
    def ended(self):        #determine if processes has ended: completed it's total execution
        if self.cpu == []:
            return True

def initializer(**args):    #initialize the processes
    if(args == {}):
        p1=process("P1",[5, 27, 3, 31, 5, 43, 4, 18, 6, 22, 4, 26, 3, 24, 4])
        p2=process("P2",[4, 48, 5, 44, 7, 42, 12, 37, 9, 76, 4, 41, 9, 31, 7, 43, 8])
        p3=process("P3",[8, 33, 12, 41, 18, 65, 14, 21, 4, 61, 15, 18, 14, 26, 5, 31, 6])
        p4=process("P4",[3, 35, 4, 41, 5, 45, 3, 51, 4, 61, 5, 54, 6, 82, 5, 77, 3])
        p5=process("P5",[16, 24, 17, 21, 5, 36, 16, 26, 7, 31, 13, 28, 11, 21, 6, 13, 3, 11, 4])
        p6=process("P6",[11, 22, 4, 8, 5, 10, 6, 12, 7, 14, 9, 18, 12, 24, 15, 30, 8])
        p7=process("P7",[14, 46, 17, 41, 11, 42, 15, 21, 4, 32, 7, 19, 16, 33, 10])
        p8=process("P8",[4, 14, 5, 33, 6, 51, 14, 73, 16, 87, 6])
        group = [p1,p2,p3,p4,p5,p6,p7,p8]
    return group

def display(group,time,waste):         #display total time, utilization thruput, tw, ttr, tr, averages
    total_time = round(time)                                    #total time to run all process
    utilization = round(((total_time-waste)/total_time)*100,2)  #prof rounded to nearest tenth place, so I did, too!
    thruput = len(group)/total_time                             #thruput = #process/total time
    print(f"Total Time:\t{total_time} units\nUtilization:\t{utilization}%\nThruput:\t{thruput}")
    a_tw = 0    #average waiting time
    a_ttr = 0   #average turnaround time
    a_tr = 0    #average response time
    num_p = len(group)  #number of processes
    print(f"Process\tTw\tTtr\tTr")
    for p in group:     #calculate sum for averages
        print(f"{p.name}\t{round(p.tw)}\t{round(p.ttr)}\t{round(p.tr)}")
        a_tw += p.tw
        a_ttr += p.ttr
        a_tr += p.tr
    a_tw /= num_p       #finish calculating averages
    a_ttr /= num_p
    a_tr /= num_p 
    print(f"Average\t{round(a_tw,2)}\t{round(a_ttr,2)}\t{round(a_tr,2)}")

def display_options(mode):          #display viewing options
    mode = mode.lower() 
    if (mode == 'regular'):
        regular = True
        extra = False 
    elif (mode == 'extra'):
        regular = True
        extra = True
    else: 
        regular = False
        extra = False
    return regular,extra

def pdata(*args):           #display queue data if not empty
    groups = args 
    print("Process\tReady\tCPU Bursts\t")
    for group in groups:
        if not (group==[]):
            if not (group[0].pr==0):
                print(f"QUEUE {group[0].pr} \t|||||||||||||||||||||||||||||||||||||||")
            else: 
                print(f"IO Queue\t|||||||||||||||||||||||||||||||||||||||")
        else: 
            print("Emmpty \t|||||||||||||||||||||||||||||||||||||||")
        for p in group:
            print(f"{p.name}\t:{p.rdy}\t{p.cpu}")

#insert in order of rdy time or job length
def insort(group,g, metric):
    if group == []:
        group.append(g)     #if empty, add g to the sorted list
    elif metric == 'rdy':   #FCFS is based on ready time
        for i in range(len(group)):
            p = group[i] 
            if (g.rdy < p.rdy): 
                group.insert(i,g)   #insert in ascending order of rdy time (time available)
                break
        else: 
            group.insert(i+1,g)
    elif metric == 'job':       #SJF is based on job length 
        for i in range(len(group)):
            p = group[i] 
            if (g.job < p.job): 
                group.insert(i,g)   #insert in ascending order of job length 
                break
        else: 
            group.insert(i+1,g)

def FCFS(group,mode='regular'):            #First Come First Serve
    print("FCFS\tFirst Come First Serve \t**********************************************************************************************")
    #in FCFS, first process that arrives gets to run 1 cpu AND 1 io burst; p# CAN run CPU during q#'s io burst.  
    regular,extra = display_options(mode)
    time = 0                #current execution time in the timing diagram
    waste = 0               #time NOT used for any cpu process bursts      
    ready = group.copy()    #all process arrive at the same time, so are all in ready queue
    io_q = []               #the not ready queue: processes in IO burst
    while not (ready==[] and io_q==[]):  #while there ARE processes to run
        if(ready == []):    #if there are NO more ready processes
            io_q[0].pr = 1  #priority for labeling
            ready.append(io_q.pop(0))   #get the earliest process from sorted io queue
            continue                    #and run it
        else:                           #otherwise
            p = ready[0]                #get the process at front of rdy queue
        if (regular):
            print(f"{p.name}************************************************************************")
            print(f"Ready Time:\t{p.rdy}\nCurrent Time:\t{time}")
        if(p.rdy > time):           #if NO processes are ready (i,e: in IO burst)
            if (extra):
                print(f"{p.name} is NOT ready yet!!!")
            waste += p.rdy-time     #record the time NOT used for any process bursts
            time = p.rdy            #skip to when process IS ready
        p.tw += time-p.rdy          #current time - the time p's been ready = the time p's been waiting
        if (extra):
            print(f"{time} to {round(time+p.cpu[0],2)} = {p.name} cpu")
        if (p.tr < 0):              #if NO response time has been assigned, then this is the first cpu burst
            p.tr = time             #record the response time
        time += p.cpu.pop(0)        #run cpu burst, add it to the time spent
        if (p.ended()):             #if process ended
            if(regular):
                print(f"{p.name} has ended.")
            p.ttr = time            #record the turnaround time
            ready.pop(0)            #remove it from the queue
            continue                #and continue to next process
        if(extra):
            print(f"{time} to {round(time+p.io[0],2)} = {p.name} io")
        p.rdy = round(time+p.io.pop(0),2)   #otherwise, run io burst & record when process will be ready
        if(extra):
            print(f"{p.name} in io queue, next cpu burst: {p.cpu[0]}")
        p.pr = 0
        insort(io_q,p,'rdy')            #add process to the io queue because it is in io. 
        if not(io_q == []) and (io_q[0].rdy <= time):  #if there ARE processes in io, add the first one who IS ready to ready q 
            io_q[0].pr = 1
            ready.append(io_q.pop(0))   #remove the ready process from io and into ready queue
        ready.pop(0)        #finally remove process that just ran from ready queue
        if (regular):
            pdata(ready,io_q)
    display(group,time,waste)   

def SJF(group,mode='regular'):             #Shortest Job First
    print("SJF\tShortest Job First \t**********************************************************************************************")
    #in SJF, we prioritize shortest available CPU burst, if not available, the the soonest available CPU burst  
    regular,extra = display_options(mode)
    time = 0                #current execution time in the timing diagram
    waste = 0               #time NOT used for any cpu process bursts      
    ready = []              #ready queue starts off empty, based on shortest job, NOT arrival time
    for g in group:
        g.pr = 1
        insort(ready,g,'job')   #insert based on job length 
    io_q = []
    while not (ready==[] and io_q==[]):  #while there are processes to run
        p = ready[0]                #get the process at front of rdy queue
        if(regular):
            print(f"{p.name}************************************************************************")
            print(f"Ready Time:\t{p.rdy}\nCurrent Time:\t{time}\nJob Length:\t{p.job}")
        if(p.rdy > time):           #if NO processes are ready (i,e: in IO burst)
            if(extra):
                print(f"{p.name} is NOT ready yet!!!")
            waste += p.rdy-time     #record the time NOT used for any process bursts
            if(extra):
                print(f"WASTE UPDATE ALERT: {waste} ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~")
            time = p.rdy        #skip to when process IS ready
        p.tw += time-p.rdy      #current time - the time p's been ready = the time p's been waiting
        if(extra):
            print(f"{time} to {round(time+p.cpu[0],2)} = {p.name} cpu")
        if (p.tr < 0):      #if NO response time has been assigned, then this is the first cpu burst
            p.tr = time     #record the response time
        time += p.cpu.pop(0)#run cpu burst, add it to the time
        if (p.ended()):     #if process ended
            if(regular):
                print(f"{p.name} has ended.")
            p.ttr = time    #record the turnaround time
            ready.pop(0)    #remove it from the queue
        else:
            p.job = p.cpu[0]    #new job length     
            if(extra):
                print(f"{time} to {round(time+p.io[0],2)} = {p.name} io")
            p.rdy = round(time+p.io.pop(0),2)   #otherwise, run io burst & record when process will be ready
            if (extra):
                print(f"{p.name} in io queue, next cpu burst: {p.cpu[0]}")
            insort(io_q,p,'rdy')    #add process to the io queue because it is in io. 
            if not (ready == []):
                ready[0].pr = 0
                ready.pop(0)        #remove process from the ready queue
        #add all ready processes to ready queue in order of shortest job
        if (not(io_q == [])):   #if there are processes in io, none ready, add the shortest one who is ready to ready q 
            i=0
            while(i < len(io_q)):   #for every process in io
                q = io_q[i] 
                if (q.rdy <= time): #if it's ready
                    q.pr = 1
                    insort(ready,io_q.pop(i),'job') #insert into rdy queue based on job length
                    i-=1
                i+=1 
        #if ready is empty, then add the soonest ready process
        if (ready == []) and (not(io_q==[])):
            io_q[0].pr = 1      #priority of rdy queue (labelling purposes)
            insort(ready,io_q.pop(0),'job')
        if(regular):
            pdata(ready,io_q)
    display(group,time,waste)

def MLFQ(group,mode=''):         #Multi-Level Queue
    print("MLFQ\tMulti-Level Frequencey Queue \t**********************************************************************************************")
    regular,extra = display_options(mode)
    queue2 = [] #RR: Tq = 10
    queue3 = [] #FCFS 
    tq1 = 5
    tq2 = 10
    #in this MQFL, we use RR:tq=5, RR:tq=10, FCFS for each queue, respectively.
    time = 0                #current execution time in the timing diagram
    waste = 0               #time NOT used for any cpu process bursts      
    ready = group.copy()    #ready queue starts off empty, based on shortest job, NOT arrival time
    io_q = []               #we only end when all queues are empty
    while not (ready==[] and queue2 == [] and queue3 == [] and io_q==[]):  #while there are processes to run
        if not (ready==[]): #if queue1 is NOT empty
            curr_q = ready  #we use it
            next_q = queue2 #and reference queue2
            p = curr_q[0]   #process at front of queue1
            tq = tq1        #use tq for queue 1
            pr = 2          #the next priority below queue 1
        elif not (queue2==[]):#use next level queue
            curr_q = queue2 #use queue2
            next_q = queue3 #reference the lower-level queue
            p = curr_q[0]   #get process from queue2
            tq = tq2        #use the tq in queue2's RR algorithm
            pr = 3          #the next priority below queue2
        elif not (queue3==[]):
            p = queue3[0]   #there is no priority below 3, so just use this queue & anything leaving queue3 goes to io
        else:   #if all queues empty
            io_q[0].pr = 1              #priority of the ready queue
            ready.append(io_q.pop(0))   #place the soonest ready process in queue1
            continue                    #and run
        if (p.pr == 1 or p.pr == 2):    #we use RR for these two queues
            if(regular):
                print(f"{p.name}************************************************************************")
                print(f"Ready Time:\t{p.rdy}\nCurrent Time:\t{time}\nJob Length:\t{p.job}")
            if(p.rdy > time):   #if NO processes are ready (i,e: in IO burst)
                if(extra):
                    print(f"{p.name} is NOT ready yet!!!")
                waste += p.rdy-time     #record the time NOT used for any process bursts
                if(extra):
                    print(f"WASTE UPDATE ALERT: {waste} ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~")
                time = p.rdy        #skip to when process IS ready
            p.tw += time-p.rdy      #current time - the time p's been ready = the time p's been waiting
            if(extra):
                print(f"{p.name}'s tw = {p.tw} = {time} - {p.rdy}!!!@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@")
            if (p.tr < 0):      #if NO response time has been assigned, then this is the first cpu burst
                p.tr = time     #record the response time 
            if (p.cpu[0]>tq):   #if process does NOT finish in time
                if(extra):
                    print(f"{time} to {round(time+tq,2)} = {p.name} cpu")
                time += tq      #this is the max time we add when executing this run
            else:               #otherwise
                if(extra):
                    print(f"{time} to {round(time+p.cpu[0],2)} = {p.name} cpu")
                time += p.cpu[0]#add the time of the process
            p.cpu[0] -= tq      #regardless, remove the time that the process used
            p.job = p.cpu[0]    #definition of job: next avaialable cpu
            if (not(io_q == [])):   #if there are processes in io, none ready, add the shortest one who is ready to ready q 
                i=0
                while(i < len(io_q)):
                    q = io_q[i] 
                    if (q.rdy <= time): #only add the ready processes from IO into queue1
                        ready.append(io_q.pop(0)) 
                        i-=1
                        q.pr = 1        #priority resets upon leaving IO
                    i+=1
            if (p.cpu[0] <= 0):     #if the process CPU burst has ended
                p.cpu.pop(0)        #remove it 
                if (p.ended()):     #if process ended
                    if(regular):
                        print(f"{p.name} has ended.")
                    p.ttr = time    #record the turnaround time
                    curr_q.pop(0)    #remove it from the queue
                else:#in IO
                    p.job = p.cpu[0]    #new job length     
                    if(extra):
                        print(f"{time} to {round(time+p.io[0],2)} = {p.name} io")
                    p.rdy = round(time+p.io.pop(0),2)   #otherwise, run io burst & record when process will be ready
                    if(extra):
                        print(f"{p.name} in io queue, next cpu burst: {p.cpu[0]}")
                    p.pr = 0            #priority to identify processes in io
                    insort(io_q,p,'rdy')    #add process to the io queue because it is in io. 
                    curr_q.pop(0)           #remove from the current queue
            else:#cpu burst not finished
                p.rdy = time
                p.job = p.cpu[0]    #new job length     
                if(extra):
                    print(f"{p.name}'s CPU' did NOT end in {tq} units. pr = {pr}")
                p.pr = pr
                next_q.append(curr_q.pop(0))        #remove process from the ready queue
        elif (p.pr == 3):
                if(regular):
                    print(f"{p.name}************************************************************************")
                    print(f"Ready Time:\t{p.rdy}\nCurrent Time:\t{time}")
                if(p.rdy > time):   #if NO processes are ready (i,e: in IO burst)
                    if(extra):
                        print(f"{p.name} is NOT ready yet!!!")
                    waste += p.rdy-time     #record the time NOT used for any process bursts
                    time = p.rdy            #skip to when process IS ready
                p.tw += time-p.rdy  #current time - the time p's been ready = the time p's been waiting
                if(extra):
                    print(f"{time} to {round(time+p.cpu[0],2)} = {p.name} cpu")
                if (p.tr < 0):     #if NO response time has been assigned, then this is the first cpu burst
                    p.tr = time     #record the response time
                time += p.cpu.pop(0)#run cpu burst, add it to the time
                if (p.ended()):     #if process ended
                    if(regular):
                        print(f"{p.name} has ended.")
                    p.ttr = time    #record the turnaround time
                    queue3.pop(0)   #remove it from the queue
                    continue        #and continue to next process
                p.job = p.cpu[0]
                if(extra):
                    print(f"{time} to {round(time+p.io[0],2)} = {p.name} io")
                p.rdy = round(time+p.io.pop(0),2)   #otherwise, run io burst & record when process will be ready
                if(extra):
                    print(f"{p.name} in io queue, next cpu burst: {p.cpu[0]}")
                p.pr = 0
                insort(io_q,p,'rdy')    #add process to the io queue because it is in io. 
                queue3.pop(0)           #remove process from the ready queue
        if(regular):
            pdata(ready,queue2,queue3,io_q)
    display(group,time,waste)
#main
button = '?' 
while not (button == '0'):
    group = initializer()
    button = input("Please Select ↓\t************************************************************************************************\n1-FCFS\n2-SJF\n3-MLFQ\n0-Exit\nYour Choice: ")
    if (button == '1' or button == '2' or button == '3'):
        view = input("2Scheduling Information Display ↓\t_______________________________________________,________________\nRegular: For grading purposes\nExtra: For debugging/additional features\nMinimal: Show as little as possible\nType Your Choice: ")
        if (button=='1'):
            FCFS(group,view)
        elif(button=='2'):
            SJF(group,view)
        elif(button=='3'):
            MLFQ(group,view)
    else:
        break

