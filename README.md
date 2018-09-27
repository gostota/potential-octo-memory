# potential-octo-memory
debugging some stuff


import oracleCon
import datetime
import parameters

def getQueryString(p):
    return """
    select a.EVENTID, a.EVENTDATETIME, a.TargetSystem,a.OPERATION,a.APPLICATION,a.MSGFLOW,a.STAGE,a.MESSAGEID,a.LOGLEVEL,a.CODE
    from IIBNRT.ESBEVENT a,IIBNRT.ESBEVENTDETAILS b
    where a.EVENTID=b.EVENTID
            and a.EVENTID like '{}'
            and a.TARGETSYSTEM like '{}' 
            and a.OPERATION like '{}' 
            and a.APPLICATION like '{}' 
            and a.MSGFLOW like '{}' 
            and a.STAGE like '{}'
            and a.LOGLEVEL like '{}'
            and a.EVENTDATETIME >= TO_DATE('{}','YYYY MM DD HH24:MI:SS')  
            and a.EVENTDATETIME <= TO_DATE('{}','YYYY MM DD HH24:MI:SS')
            """.format(p.eventId,p.targetSystem,p.operation,p.application,p.msgFlow,p.stage,p.logLevel,p.startTime.strftime("%Y %m %d %H:%M:%S"),p.endTime.strftime("%Y %m %d %H:%M:%S"))

def switch(input,value):
    d={
            '1': "and a.EVENTID like '"+value+"'",
            '2': "and a.TARGETSYSTEM like '"+value+"'",
            '3': "and a.OPERATION like '"+value+"'",
            '4': "and a.APPLICATION like '"+value+"'",
            '5': "and a.MSGFLOW like '"+value+"'",
            '6': "and a.STAGE like '"+value+"'",
            '7': "and a.LOGLEVEL like '"+value+"'",
            '8': "and b.CODE like '"+value+"'",
            '9': "and b.MESSAGECCSID like '"+value+"'",
        }
    return d[input]


def getDBInfo(oracle):
    print("ESBEVENT table columns with content examples ")
    colNames=oracle.execute("SELECT column_name FROM all_tab_cols where table_name='ESBEVENT' and not column_name like 'SYS_%'")
    for i in colNames:
        print("\t"+i[0],end=":")
        objNames=oracle.execute("SELECT distinct "+i[0]+" from IIBNRT.ESBEVENT FETCH FIRST 5 ROWS ONLY")
        print (objNames)
        print("\n\n")

def getTextOutput(oracle,data):
    f=open("output\DBoutput.txt","+w")
    temp=''
    colNames=oracle.execute("SELECT column_name FROM all_tab_cols where (table_name='ESBEVENT' or table_name='ESBEVENTDETAILS') and not column_name like 'SYS_%'")
    for i in range(0,len(colNames)):
        temp+=("%40s" % colNames[len(colNames)-i-1][0])
    f.write(temp)
    for i in data:
        temp='\n'
        for j in range(0,len(i)):
            try:
                temp+=("%40s" % i[j])
            except TypeError:
                temp+=i[j].strftime("%Y %m %d %H:%M:%S")
        f.write(temp)
    f.close()

#Iinterface
oracle=oracleCon.oracleCon()
oracle.connect()

p=parameters.parameters()
mainMenu={
    1:"Which one of the following parameters u want to change? 0:noone 1:EventID 2:TargetSysstem 3:Operation 4:Application 5:Msgflow 6:Stage 7:LogLevel 8:Code 9:MESSAGECCSID\n",
    2:"Print messageId:"}
inp=input("type:\n1 to start searching\n2 to get transaaction via messageid\n3 to show database info\n")
if inp=='1':
    print(inp)
    p=getQueryString(p)
    paramInput=input(mainMenu[1])
    while paramInput!='0':
        p+=switch(paramInput,input("Type required value\n"))
        paramInput=input("Which one of the following parameters u want to change? 0:noone 1:EventID 2:TargetSysstem 3:Operation 4:Application 5:Msgflow 6:Stage 7:LogLevel 8:Code 9:MESSAGECCSID\n")
    getTextOutput(oracle,oracle.execute(p))
elif inp=='2':
    print(mainMenu[2])
    print(oracle.execute("SELECT * from IIBNRT.ESBEVENT a,IIBNRT.ESBEVENTDETAILS b where a.EVENTID=b.EVENTID where a.messageid='{}'").format(input()))
elif inp=='3':
    getDBInfo(oracle)

oracle.disconnect()
