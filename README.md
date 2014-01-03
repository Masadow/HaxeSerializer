HaxeSerializer
==============

Macro helper for `haxe.Serialize` and `haxe.Unserialize`

Its goal is to help you to serialize and unserialize your custom class

	class Test implements ISerializable
	{
		@serialize public var serialized : Int = 0;
		public var notSerialized : Int = 0;
	}

	var t : Test = new Test();
	t.serialized = 1;
	t.notSerialized = 1;
	var s = haxe.Serialize(t);
	var tbis : Test = haxe.Unserialize(t);
	trace(tbis.serialized); // 1
	trace(tbis.notSerialized); // 0

If you try to serialize the following class with `haxe.Serialize`, only `serialized` will be outputed.

Here is a full usage sample from a personal project

	package ;
	import flash.Lib;
	import flash.Memory;
	import flash.system.Capabilities;
	import flash.utils.ObjectOutput;
	import flixel.util.FlxSave;
	import haxe.Serializer;
	import haxe.Unserializer;
	import map.Level;
	import utils.ISerializable;
	
	/**
	 * ...
	 * @author Masadow
	 */
	class Save implements ISerializable
	{
		private static var shared : FlxSave = null;
		
		@serialize public var money : Int;
		@serialize public var power : Int;
		@serialize public var powerPrice : Int;
		@serialize public var firerate : Float;
		@serialize public var fireratePrice : Int;
		@serialize public var target : Int;
		@serialize public var targetPrice : Int;
		@serialize public var level(default, null) : Array<Level>;
	
		public function new() 
		{
			init();
			flush(false);
		}
		
		private function init()
		{
			money = 0;
			power = 20;
			powerPrice = 200;
			firerate = 0.5;
			fireratePrice = 300;
			target = 1;
			targetPrice = 1000;
	
			//Reset the first lvl
			level = new Array<Level>();
			level.push(new Level(1));
		}
		
		public static function load(Create : Bool = true) : Save
		{
			if (shared == null)
			{
				shared = new FlxSave();
				shared.bind("save");
				//shared.erase();
				//shared.bind("save");
				if (Create && (!Std.is(shared.data.content, String) || cast(shared.data.content, String).length == 0))
				{
					new Save();
				}
			}
			trace(Unserializer.run(shared.data.content));
			return Unserializer.run(shared.data.content);
		}
		
		public function flush(Fast : Bool = true)
		{
			shared.data.content = Serializer.run(this);
			shared.flush();
			
			if (!Fast)
			{
				shared.close();
				shared.destroy();
				shared = null;
				load(false);
			}
		}
		
		public function reset()
		{
			shared.erase();
			shared.close();
			Save.load();
		}
		
	}