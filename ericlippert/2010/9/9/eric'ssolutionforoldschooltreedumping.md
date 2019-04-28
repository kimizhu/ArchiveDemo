# Eric's solution for old school tree dumping

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/9/2010 6:58:00 AM

-----

 

sealed class Dumper  
{  
    private StringBuilder sb;  
    static public string Dump(Node root)  
    {  
        var dumper = new Dumper();  
        dumper.Dump(root, "");  
        return dumper.sb.ToString();  
    }  
    private Dumper()  
    {  
        this.sb = new StringBuilder();  
    }  
    private void Dump(Node node, string indent)  
    {  
        // Precondition: indentation and prefix has already been output  
        // Precondition: indent is correct for node's \*children\*  
        sb.AppendLine(node.Text);  
        for (int i = 0; i \< node.Children.Count; ++i )  
        {  
            bool last = i == node.Children.Count - 1;  
            var child = node.Children\[i\];  
            sb.Append(indent);  
            sb.Append(last ? '└' : '├');  
            sb.Append('─');  
            Dump(child, indent + (last ? "  " : "│ "));  
        }  
    }  
}

