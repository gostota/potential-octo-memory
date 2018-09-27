import oracleCon
import datetime

def getQueryString():
    startTime=datetime.datetime(2018, 8, 2, 16, 0, 1, 522000)
    endTime=datetime.datetime(2018, 8, 2, 17, 0, 1, 522000)
    return """
    select a.EVENTID, a.EVENTDATETIME, a.TargetSystem,a.OPERATION,a.APPLICATION,a.MSGFLOW,a.STAGE,a.MESSAGEID,a.LOGLEVEL,a.CODE
    from IIBNRT.ESBEVENT a,IIBNRT.ESBEVENTDETAILS b
    where a.EVENTID=b.EVENTID
            and a.EVENTDATETIME >= TO_DATE('{}','YYYY MM DD HH24:MI:SS')  
            and a.EVENTDATETIME <= TO_DATE('{}','YYYY MM DD HH24:MI:SS')
            """.format(startTime.strftime("%Y %m %d %H:%M:%S"),endTime.strftime("%Y %m %d %H:%M:%S"))

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
            '10': "and utl_raw.cast_to_varchar2(dbms_lob.substr(b.MESSAGE,2000,1)) like '%"+value+"%'",           
            '11': "and a.MESSAGEID like '%"+value+"%'"
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

mainMenu={
    1:"Which one of the following parameters u want to change? 0:noone 1:EventID 2:TargetSysstem 3:Operation 4:Application 5:Msgflow 6:Stage 7:LogLevel 8:Code 9:MESSAGECCSID\n",
    2:"Print messageId:"}
inp=input("type:\n1 to start searching\n2 to get transaaction via messageid\n3 to show database info\n")
if inp=='1':
    p=getQueryString()
    paramInput=input(mainMenu[1])
    while paramInput!='0':
        temp=input("Type required value\n")
        if temp!="SPECSIMVOL":
            p+=switch(paramInput,temp)
            paramInput=input("Which one of the following parameters u want to change? 0:noone 1:EventID 2:TargetSysstem 3:Operation 4:Application 5:Msgflow 6:Stage 7:LogLevel 8:Code 9:MESSAGECCSID 10:search substr in message\n")
        else: 
            p+=switch(paramInput+100,temp)
            paramInput=input("Which one of the following parameters u want to change? 0:noone 1:EventID 2:TargetSysstem 3:Operation 4:Application 5:Msgflow 6:Stage 7:LogLevel 8:Code 9:MESSAGECCSID 10:search substr in message\n")
    getTextOutput(oracle,oracle.execute(p))
elif inp=='2':
    #Здесь нужно дописать как в первом случае
    p=getQueryString()
    p+=switch('11',input(mainMenu[2]))
    getTextOutput(oracle,oracle.execute(p))
elif inp=='3':
    getDBInfo(oracle)

oracle.disconnect()
