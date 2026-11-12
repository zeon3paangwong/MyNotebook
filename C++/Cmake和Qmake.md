Qamke构建动态库后需要LIBS +=，LIBS后还需要INCLUDE_PATH把头文件包含进来。LIBS后会在相关文件夹搜索dll文件exe文件夹，lib文件夹，环境变量。

Cmake把构建动态库的目录add_subdirectory后就不用像qmake一样进行单独链接LIBS +=

部分三方库编译库文件只需要对应的C/cpp文件，不需要头文件，比如说modbus库，而Qt相关的库需要头文件：因为 AUTOMOC ON 要扫描头文件里的 Q_OBJECT，且 install(FILES ${ads_HEADERS}) 要用这个列表装头文件。