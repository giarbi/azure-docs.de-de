Sie können mit Azure Resource Manager Parameter für die Werte, die beim Bereitstellen der Vorlage verwendet werden, definieren. Die Vorlage enthält einen Abschnitt namens `parameters`, der alle Parameterwerte enthält. Jeder Parameterwert wird in der Vorlage verwendet, um die bereitgestellten Ressourcen zu definieren.

> [!NOTE]
> Definieren Sie keine Parameter für Werte, die sich nicht ändern. Definieren Sie Parameter nur für Werte, die abhängig vom bereitzustellenden Projekt oder abhängig von der Zielumgebung für die Bereitstellung variieren.

Folgendes gilt beim Definieren von Parametern:

* Verwenden Sie das Feld **allowedValues**, um anzugeben, welche Werte Benutzer während der Bereitstellung angeben können.

* Verwenden Sie das Feld **defaultValue**, um dem Parameter Standardwerte zuzuweisen, falls im Rahmen der Bereitstellung kein Wert angegeben wurde. 
