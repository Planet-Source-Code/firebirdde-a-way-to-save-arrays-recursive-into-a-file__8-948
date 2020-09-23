<div align="center">

## A way to save arrays recursive into a file


</div>

### Description

This little code snipplet can save and reload arrays recursive into/from a file. You can use this if you want to add a guestbook to your page and there's no MySql database left, for example.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[FirebirdDE](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/firebirdde.md)
**Level**          |Intermediate
**User Rating**    |4.0 (8 globes from 2 users)
**Compatibility**  |PHP 3\.0, PHP 4\.0
**Category**       |[Data Structures](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/data-structures__8-8.md)
**World**          |[PHP](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/php.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/firebirdde-a-way-to-save-arrays-recursive-into-a-file__8-948/archive/master.zip)





### Source Code

```
<?php
	/*
		Save arrays recursive into a file
		Construct of a file:
		\0{ keyname\0value\0keyname\0value\0keyname\0\0{SUBARRAY\0}\0}
		Syntax:
		SaveArrayToFile(Handle of a BINARY opened file, an array);
		LoadArrayFromFile(Handle of a BINARY opened file);
		NOTE: Its necessary to open the file
		 in binary mode if you're using M$ Windows.
		 Use "rb" and "wb" to open a file in binary mode.
		 Script by Firebird, www.berndt-cpk.de
		 Sorry for possible grammar faults, I'm no native speaker :)
	*/
	// Save the array
	// Syntax: SaveArrayToFile(Handle of a BINARY opened file, an array);
	function SaveArrayToFile($vFile, $vArray)
	{
		// Every array starts with chr(1)+"{"
		fwrite($vFile, "\0{");
		// Go through the given array
		reset($vArray);
		// Start a loop. One could put the "next" command here, but if an
		// entry was empty, "next" would return false.
		while (true)
		{
			// Get the current "record" of the array
			$Current = current($vArray);
			// Get the current key. I use addshashes cause the key could
			// contain \0, and this would be interprated as End-Of-Record
			$MyKey = addslashes(strval(key($vArray)));
			// Is it a sub-array?
			if (is_array($Current)) {
				// Save the key into the file
				fwrite($vFile, $MyKey."\0");
				// Call me (the function) using the sub array
				SaveArrayToFile($vFile,$Current);
				// Write the record delemitter.
				fwrite($vFile, "\0");
			} else {
				// Save the record into the file
				$Current = addslashes($Current);
				fwrite($vFile,"$MyKey\0$Current\0");
			}
			// Proceed to the next record. Skip Empty records
			++$i;
			while (!next($vArray))
			{
				if (++$i > count($vArray)) break;
			}
			if ($i > count($vArray)) break;
		}
		// Close current array
		fwrite($vFile,"\0}");
	}
	// Load an array
	// Syntax: LoadArrayFromFile(Handle of a BINARY opened file);
	function LoadArrayFromFile($vFile)
	{
		// Create empty array
		$ForRet = array();
		// Does the file contain an array?
		$Wert = fread($vFile,2);
		if ($Wert != "\0{") return;
		// Again, start a loop
		while (true) {
			// Does the array end here?
			if (NextMatches($vFile,"\0}")) {
				// Read in the closer-string, otherwise the function would fail.
				fread($vFile,2);
				// Return the array
				return $ForRet;
			}
			// Get the key name
			$MyKey = "";
			while (true) {
				$Zeichen = fread($vFile,1);
				if ($Zeichen == "\0")
					break;
				else
					$MyKey .= $Zeichen;
			}
			// Remove slashes
			$MyKey = stripslashes($MyKey);
			// Is it a sub-array ?
			if (NextMatches($vFile,"\0{")) {
				// It is a subarray ^^
				$ForRet[$MyKey] = LoadArrayFromFile($vFile);
				// Skip the delemitter
				fread($vFile,1);
			} else {
				// Read the value
				$MyVal = "";
				while (true) {
					$Zeichen = fread($vFile,1);
					if ($Zeichen == "\0")
						break;
					else
						$MyVal .= $Zeichen;
				}
				// Parse the value into the array
				$MyVal = stripslashes($MyVal);
				$ForRet[$MyKey] = $MyVal;
			}
		// Continue
		}
	}
	// Check if $Text is @ cursor position in $vFile
	// Syntax: NextMatches($vFile, $Text);
	function NextMatches($vFile, $Text)
	{
		// Save the current position in the file
		$PrevPos = ftell($vFile);
		// How long is $Text ?
		$Jump = strlen($Text);
		// Check if the file is long enaugh
		$stats = fstat($vFile);
		if (ftell($vFile) + $Jump > $stats[7])
			return false;
		// Read out a string as long as $Text
		$Erg = fread($vFile,$Jump);
		// Continue to the previous position.
		// I don't use whence for compatibility with PHP < 4
		fseek($vFile, $PrevPos);
		return ($Erg == $Text);
	}
	/*
	An example:
	Create an recursive array and save it. Reload it again and give it out.
	----------------
	*/
	$MyGuestbook = array();
	$MyGuestbook[1]['Name'] = "Test 1";
	$MyGuestbook[1]['Test'][1][1] = "Test 1.2.1";
	$MyGuestbook[1]['Test'][2] = "Test 1.3";
	$MyGuestbook[2]['Name'] = "Test 4";
	$MyGuestbook[3]['Test'] = "Test 5";
	$MyGuestbook[4] = "\0";
	for ($i=0; $i<15; $i++)
		$MyGuestbook['Subarray']['Verybig'][$i] = strval($i);
	$Datei = fopen("Test.txt","wb");
	SaveArrayToFile($Datei, $MyGuestbook);
	fclose($Datei);
	$Datei = fopen("Test.txt","rb");
	$My = LoadArrayFromFile($Datei);
	echo($My[1]['Name']."<br>");
	echo($My[1]['Test'][1][1]."<br>");
	echo($My[1]['Test'][2]."<br>");
	echo($My[2]['Name']."<br>");
	echo($My[3]['Test']."<br>");
	echo("6:: ".$My['Subarray']['Verybig'][6]."<br>");
	echo($MyGuestbook[4]=="\0" ? "The last value is chr(0)" : "Sorry, my fault");
	fclose($Datei);
	unlink("Test.txt");
	/*
	---------------------
	*/
?>
```

