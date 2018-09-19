/* by @erandelax 18/09/18:   Модуль настраиваемых горячих клавиш и команд чата
////////////////////////////////////////////////////////////////////////////////

Клиентские настройки хранятся в папке клиента в файле <Commands.txt> (описание
по формату конфигурации там же). Если файл настроек пустой или отсутствует,
клиент генерирует новый.

Для регистрации собственных команд (которые впоследствие могут будут привязаны
к команде чата или горячей клавише) воспользуйтесь следующими методами в коде
инициализации своего модуля:
```

#include "cmd_h.fos" // загрузит CmdRegister и CmdRegisterArgument (см. ниже)

// Регистрация RunServerScriptUnsafe-функции
CmdRegister("module@unsafe_FuncName","MY:COMMAND",ARG_TYPE,ARG_TYPE,ARG_TYPE);

// Регистрация серверной RunServerScript-функции
CmdRegister("module@_SafeFuncName","MY:COMMAND",ARG_TYPE,ARG_TYPE,ARG_TYPE);

// Регистрация клиентской RunClientScript-функции - внимание, она должна 
// быть объявлена либо импортирована в тот же модуль, откуда производится
// регистрация (т.е. присутствовать в пространстве имен)!
CmdRegister(@__ClientFuncName,"MY:COMMAND",ARG_TYPE,ARG_TYPE,ARG_TYPE);

// Регистрация своего подтипа аргумента для команды MY:COMMAND (при встрече в
// <Commands.txt> парсер заменит её на соответствующий <int> и пошлет в функцию.
CmdArgument("MY:COMMAND","TAG",1); // TAG => 1

```
В них:
	MY:COMMAND - тег команды, по которой она будет вызываться в <Commands.txt>
	ARG_TYPE - типы аргументов, которые можно передать функции (см. ниже),
		можно перечислять сразу несколько через |
	@__ClientFuncName - референс функции __ClientFuncName (вводить без кавычек!)

Примеры объявлений:
```
CmdRegister( // Зарегистрировать команду
	@__CritterMove, // С функции __CritterMove
	"CRITTER:MOVE", // С тегом CRITTER:MOVE
	ARG_CRITTER|ARG_CRITTERS, // Первый арг. - Critter.Id или массив таковых
	ARG_HEX|ARG_CRITTER|ARG_ITEM|ARG_CUSTOM, // гекс, криттер, айтем или кастом
	ARG_INT|ARG_VOID // число, либо ничего (aka не обязательный аргумент)
);
CmdArgument("CRITTER:MOVE","NORTH",0);
```

Допустим, в commands.fos указано:
	@ W, CRITTER:MOVE, SELF, UP, 10

При нажатии клавиши W будет вызвана CRITTER:MOVE с ID криттера игрока в первом
аргументе, UP (0) втором и 10 в третьем. Допустим, хоткей запустил игрок, чей
криттер имеет Id == 12, тогда функция __CritterMove получит:
	__CritterMove(ARG_CRITTER,ARG_CUSTOM,ARG_INT,null,{12,0,10});

Аналогично, для инструкции 
	@ CTRL+A, CRITTER:MOVE, GROUP, TARGET:CRITTER
(<ARG_CRITTERS> GROUP и <ARG_CRITTER> CRITTER, подробнее в <Commands.txt>)

Функция __CritterMove получит:
	__CritterMove(ARG_CRITTERS,ARG_CRITTER,ARG_VOID,{
		3, // Количество id в ARG_CRITTERS
		500013, // id первого криттера группы ...
		500015,
		500018,
		203, // id криттера 
	});

Чтобы не заморачиваться с парсингом аргументов можно воспользоваться макросами
из этого файла:



//////////////////////////////////////////////////////////////////////////////*/

// Тип аргумента (ARG_TYPE из примеров выше)
#define ARG_VOID 				(0) 	// Аргумент опционален/отсутствует
#define ARG_INT 				(1) 	// Произвольное число <int>
#define ARG_INTS 				(1<<1)  // Массив чисел <int>
#define ARG_STRING 				(1<<2) 	// Произвольная строка <string>
#define ARG_CRITTER 			(1<<3)  // ID одного криттера <int>
#define ARG_CRITTERS 			(1<<4) 	// Массив из ID криттеров <int[]>
#define ARG_ITEM 				(1<<5)  // ID одного итема <int>
#define ARG_ITEMS 				(1<<6)  // Массив из ID итемов <int[]>
#define ARG_HEX					(1<<7)  // Координаты гекса <uint16[](2)>
#define ARG_PATH 				(1<<8)  // Координаты двух гексов <uint16[](4)>
#define ARG_CUSTOM 				(1<<9)  // (см. <CmdArgument> выше) <int>

///////////////////////////////////////////////////////////////////////////////
// Макросы для быстрого объявления Client/Safe/Unsafe-функций и п/а аргументов
///////////////////////////////////////////////////////////////////////////////
#ifdef __CLIENT
#define ClientCommand(FuncName) void ##FuncName##(int _arg0, int _arg1, int _arg2, string@ _argS, int[]@ _argI)
#endif

#ifdef __SERVER
#define ServerUnsafeCommand(FuncName,Sender) void ##FuncName##(Critter& Sender, int _arg0, int _arg1, int _arg2, string@ _argS, int[]@ _argI)
#define ServerCommand(FuncName,Sender) void ##FuncName##(Critter& player, int _arg0, int _arg1, int _arg2, string@ _argS, int[]@ _argI)
#endif

#define ParseArguments(Arguments) CmdArgumentParser@ Arguments = CmdArgumentParser(int _arg0, int _arg1, int _arg2, string@ _argS, int[]@ _argI)

/* Пример использования:

ClientCommand(__move_critter){ 
	ParseArguments(Arguments);
	
	// Move critters group to critter
	if( Arguments.Match(ARG_CRITTERS,ARG_CRITTER,ARG_VOID) ){ 
		Critter@[] critters = Arguments.First;
		// ...
	} else if ( Arguments.Match( ... ) ) ...

}

Преобразуется в:
void __move_critter(int _arg0, int _arg1, int _arg2, string@ _argS, int[]@ _argI){
	CmdArgumentParser@ Arguments = CmdArgumentParser(int _arg0, int _arg1, int _arg2, string@ _argS, int[]@ _argI);

 	// Move critters group to critter
	if( Arguments.Match(ARG_CRITTERS,ARG_CRITTER,ARG_VOID) ){
		Critter@[] critters = Arguments.First;
		// ...
	} else if ( Arguments.Match( ... ) ) ...
}

Больше примеров - в <cmd.fos>.
*/

///////////////////////////////////////////////////////////////////////////////
// Парсер аргументов отдельной команды (внутри Safe/Unsafe/Client-обработчиков)
///////////////////////////////////////////////////////////////////////////////
/* 

CmdArgumentParser@ Arguments = CmdArgumentParser(arg0,arg1,arg2,argString,argInts);
if( Arguments.Match(ARG_CRITTER,ARG_HEX,ARG_STRING) ){
	Critter@ cr = Arguments.First;
	int[] hex =  Arguments.Second;
	string name = Arguments.Third;
}

*/

// Хранилище аргументов <int|int[]|Critter|CritterCl|Item|ItemCl|string>
shared class CmdValue
{
	any storage;

	CmdValue@ opAssign(string value){ // SET ARG_STRING
		this.storage.store(value);
		return this;
	}
	CmdValue@ opAssign(int value){ // SET ARG_INT|ARG_STRING|ARG_CRITTER|ARG_ITEM|ARG_CUSTOM
		this.storage.store(value);
		return this;
	}
	CmdValue@ opAssign(int[] values){ // SET ARG_INTS|ARG_CRITTERS|ARG_ITEMS
		this.storage.store(values);
		return this;	
	}
	CmdValue@ opAssign(uint16[] values){ // SET ARG_HEX|ARG_PATH
		this.storage.store(values);
		return this;		
	}
	string opImplConv(){ // SET ARG_STRING
		string value;
		this.storage.retrieve(value);
		return value;
	}
	int opImplConv() { // GET ARG_INT|ARG_STRING|ARG_CRITTER|ARG_ITEM|ARG_CUSTOM
		int value;
		this.storage.retrieve(value);
		return value;
	}
	int[] opImplConv() { // GET ARG_INTS|ARG_CRITTERS|ARG_ITEMS
		int[] value;
		this.storage.retrieve(value);
		return value;
	}
	uint16[] opImplConv() { // GET ARG_HEX|ARG_PATH
		uint16[] value;
		this.storage.retrieve(value);
		return value;
	}
	#ifdef __SERVER
	Critter@ opImplConv() { // GET ARG_CRITTER
		int value;
		this.storage.retrieve(value);
		return GetCritter(value);
	}
	Item@ opImplConv() { // GET ARG_ITEM
		int value;
		this.storage.retrieve(value);
		return GetItem(value);	
	}
	Critter@[] opImplConv() { // GET ARG_CRITTERS
		Critter@[] result;
		int[] value;
		this.storage.retrieve(value);
		for(uint pos = 0, len = value.length; pos < len; pos++) 
			result.insertLast(GetCritter(value[pos]));
		return result;
	}
	Item@[] opImplConv() { // GET ARG_ITEMS
		Item@[] result;
		int[] value;
		this.storage.retrieve(value);
		for(uint pos = 0, len = value.length; pos < len; pos++) 
			result.insertLast(GetItem(value[pos]));
		return result;
	}
	#endif
	#ifdef __CLIENT
	CritterCl@ opImplConv() { // GET ARG_CRITTER
		int value;
		this.storage.retrieve(value);
		return GetCritter(value);
	}
	ItemCl@ opImplConv() { // GET ARG_ITEM
		int value;
		this.storage.retrieve(value);
		return GetItem(value);	
	}
	CritterCl@[] opImplConv() { // GET ARG_CRITTERS
		CritterCl@[] result;
		int[] value;
		this.storage.retrieve(value);
		for(uint pos = 0, len = value.length; pos < len; pos++) 
			result.insertLast(GetCritter(value[pos]));
		return result;
	}
	ItemCl@[] opImplConv() { // GET ARG_ITEMS
		ItemCl@[] result;
		int[] value;
		this.storage.retrieve(value);
		for(uint pos = 0, len = value.length; pos < len; pos++) 
			result.insertLast(GetItem(value[pos]));
		return result;
	}
	#endif
}

shared class CmdArgumentParser
{
	protected CmdValue[] Arg = {,,};
	protected int[] ArgTypes = {ARG_VOID,ARG_VOID,ARG_VOID};
	// Геттеры
	#ifdef __SERVER
	
	#endif
	// Конструктор парсит аргументы
	CmdArgumentParser(int arg0, int arg1, int arg2, string@ argS, int[]@ argI){
		// Сохраняем типы аргументов
		this.ArgTypes[0] = arg0;
		this.ArgTypes[1] = arg1;
		this.ArgTypes[2] = arg2;
		int iPos = 0; // Текущая позиция в argI
		int sPos = 0; // Текущая позиция в argS
		// Парсим каждый из аргументов из соответствующих массивов
		for( uint pos = 0; pos < 3; pos++ ) switch( this.ArgTypes[pos] ){

			// Следующие типы аргументов занимают 1 <int> и копируются "как есть"
			case ARG_INT:
			case ARG_CUSTOM:
			case ARG_ITEM:
			case ARG_CRITTER: {
				this.Arg[pos] = argI[iPos++];
				} break;

			// Строки - для каждой берется длина из argI и парсится подстрока с таковой из argS
			case ARG_STRING: {
				int length = argI[iPos++]; // Получаем длину строки
				this.Arg[pos] = substring(argS, sPos, length); // Вычленяем строку
				sPos += length;
				} break;

			// <int>-массивы - сначала идет длина массива, после - элементы
			case ARG_INTS:
			case ARG_CRITTERS:
			case ARG_ITEMS: {
				int length = argI[iPos++]; // Получаем длину массива для изъятия
				int[] arr;
				for(int endPos = iPos+length; iPos < endPos; iPos++){
					arr.insertLast( argI[iPos] );
				}
				this.Arg[pos] = arr;
				} break;

			// Координаты - два гекса подряд		
			case ARG_HEX: {
				uint16[] arr = { uint16(argI[iPos++]), uint16(argI[iPos++]) };
				this.Arg[pos] = arr;
				} break;

			// Путь - две координаты подряд
			case ARG_PATH: {
				uint16[] arr = { uint16(argI[iPos++]), uint16(argI[iPos++]), uint16(argI[iPos++]), uint16(argI[iPos++]) };
				this.Arg[pos] = arr;
			} break;

			default: continue; // ARG_VOID, отсутствующий аргумент
		}
	}
	// Возвращает True, если паттерн типов аргументов совпадает с предложенным
	bool Match(int arg0, int arg1, int arg2){
		return this.ArgTypes[0] & arg0 + this.ArgTypes[1] & arg1 + this.ArgTypes[2] & arg2 == 3; 
	}
	// Получение аргументов
	CmdValue@ opIndex(uint index){ // <=Arguments[0]>
		return this.Arg[index];
	}
	CmdValue@ get_First(){ // <=Arguments.First>
		return this.Arg[0];
	}
	CmdValue@ get_Second(){ // <=Arguments.Second>
		return this.Arg[1];
	}
	CmdValue@ get_Third(){ // <=Arguments.Third>
		return this.Arg[2];
	}
}

///////////////////////////////////////////////////////////////////////////////
// Импорт функций (для подключения в другие модули через <#import "cmd_h.fos">)
///////////////////////////////////////////////////////////////////////////////

#ifdef __CLIENT
	#ifndef __CMD__
		funcdef void ClientScriptFunction(int,int,int,string@,int[]@);

		import void CmdRegister(string,ClientScript@,int,int,int);
		import void CmdRegister(string,string&,int,int,int);
		import void CmdRegisterArgument(string,string,int);
	#endif
#endif

///////////////////////////////////////////////////////////////////////////////
