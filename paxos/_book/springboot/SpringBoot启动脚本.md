```shell
#!/bin/sh
echo "-------------------------start-------------------------"

if [[ $# == 1 ]] ; then
    echo "USAGE: $0 JAR_FILE.jar"
    echo "-------------------------model one-------------------------"
  elif [[ $# == 2 ]]; then
    echo "USAGE: $0 JAR_FILE.jar ARGS1"
    echo "-------------------------model two-------------------------"
  elif [[ $# == 3 ]]; then
     echo "USAGE: $0 JAR_FILE.jar ARGS1 ARGS2"
     echo "-------------------------model three-------------------------"
  else
    echo "NOT SUPPORT"
    echo "-------------------------example-------------------------"
    echo "USAGE: $0 JAR_FILE.jar"
    echo "USAGE: $0 JAR_FILE.jar ARGS1"
    echo "USAGE: $0 JAR_FILE.jar ARGS1 ARGS2"
    echo "-------------------------exit-------------------------"
    exit 1;
fi

version=$("java" -version 2>&1 | awk -F '"' '/version/ {print $2}')
jdk_home="no"
if [[ "$version" > "1.8" ]]; then
       echo default jdk version is ok,continue
       jdk_home=${JAVA_HOME}
else
      echo "default jdk is not ok, please input the path where you want to search, / is for all path"
      read jdk_path
      while [ ! -d "$jdk_path"  ]
      do
           echo "Installer: path not exist, please input again"
           read jdk_path
      done
      echo "Installer: begin to look correct jdk path...."
      for path in `find $jdk_path -name jmap`
      do
         _java=${path%/*}/java
         version=$("$_java" -version 2>&1 | awk -F '"' '{print $2}')
         if [[ "$version" > "1.8" ]]; then
        jdk_home=${_java%/bin*}
        echo "Installer: find out correct jdk,jdk home is $jdk_home"
        break
         fi
       done
fi

if [ "$jdk_home" == "no" ] ;then
  echo "Installer: no correct jdk was found,which is required jdk1.8"
  exit 0
fi

JAVA_HOME=$jdk_home
CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
echo "-------------------------java info-------------------------"
echo $(java -version)
echo "-------------------------pwd-------------------------"
echo $(pwd)
echo "-------------------------loading-------------------------"
echo "we will start [$1]"

if [[ $# == 1 ]] ; then
       echo "by [nohup java -server -Xms128m -Xmx256m -Xss256k  -jar $(pwd)/$1 1>/dev/null 2>&1 &]"
       read -p "press any key to continue"
       nohup java -server -Xms128m -Xmx256m -Xss256k  -jar $(pwd)/$1 1>/dev/null 2>&1 &
  elif [[ $# == 2 ]]; then
       echo "by [nohup java -server -Xms128m -Xmx256m -Xss256k  -jar $(pwd)/$1 $2 1>/dev/null 2>&1 &]"
       read -p "press any key to continue"
       nohup java -server -Xms128m -Xmx256m -Xss256k  -jar $(pwd)/$1 $2 1>/dev/null 2>&1 &
  elif [[ $# == 3 ]]; then
       echo "by [nohup java -server -Xms128m -Xmx256m -Xss256k  -jar $(pwd)/$1 $2 $3 1>/dev/null 2>&1 &]"
       read -p "press any key to continue"
       nohup java -server -Xms128m -Xmx256m -Xss256k  -jar $(pwd)/$1 $2 $3 1>/dev/null 2>&1 &
  else
       echo "NEVER HAPPEN"
       exit 1;
fi

echo "-------------------------all done-------------------------"




```
