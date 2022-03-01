# FakeDBCSV
FakeDBCSV pt-BR By Gilberto Shimokawa Falcão

Solução paliativa (Only avaliable on C# - sharp)- Quer criar um sisteminha rápido ou simplesmente fazer uns testes e salvar umas tabelinhas sem muito problema? Sem ter que ficar levantando um banco de dados para fazer uns testes? Então o FakeDBCSV é para você!

Por ser feito para CSV abre em qualquer computador, até utilizando bloco de notas! O que facilita muito caso não tenha o excel.

Sistema consegue tranformar qualquer objeto com atributos publicos em atributos de tabela para CSV e recuperar eles. Também suporta uso de expressão Lambda nos Selects e Deletes.

Possui sobrecarga no Insert e Update para flexibilizar uso para objetos únicos ou em massa.

Permite fazer chaveamento de BD diretamente na FakeTable.

Requer criar um int FakeId {get; set;} para funcionamento do sistema. Posterioremente pode exportar as "Tabelinhas" para outro local e remover o mesmo tudo com linha de comando e depois alimentar uma Base de dados real. Possui comando de auto destruição da colmeia ao término de uso caso não queira deixar vestígios.

Controle múltiplas tabelas de forma simultânea, clone tabelas.

Alters e Drops inclusos + funções especiais! =D

Tem exemplo de uso incluso .cs

FakeDBCSV - Baixou, Importou, usou!

**EXEMPLO DE USO:**

            //Crie a instancia da Colmeia onde ficará as pastas que simulam o banco de dados.
            FakeDB FakeDB = new FakeDB();

            //Crie a subpasta, ela da a idéia de um banco de dados local, onde ficam os CSVs.
            FakeDB.CREATEDB("TESTE");

            //Você pode criar mais de um diretório.... Exceto com nomes repetidos, senão gera excessão.
            FakeDB.CREATEDB("NextBD");

            //Você pode ver todas pastas existentes diretamente no console
            FakeDB.SHOWDBS();

            //Caso precise desenvolver alguma função mais robusta, pode recuperar todos nome dessas subPastas.
            List<string> NomeDbs = FakeDB.GIVE_ME_DB_NAMES();
            NomeDbs.ForEach(x => Console.WriteLine(x));

            //Informe para FakeDB qual subpasta será utilizada.
            FakeDB.USEDB("TESTE");

            //Você pode criar qualquer tabela de objetos, basta instanciar e tipar a tabela do seu objeto favorito, e passar para ela o parametro de seu BD.
            FakeTable<SampleObject> SAMPLETABLE = new FakeTable<SampleObject>(FakeDB);

            //Crie a tabela, se possível circunde-a num try catch para não se deparar com um estouro!
            try { SAMPLETABLE.CREATE(); } catch { }

            //Insira objetos na tabela, você pode criar instancias em tempo de execução, entretanto, tais objetos necessitam de ter um atributo inteiro de nome FakeID escrito dessa forma. A posição é indiferente.
            //Lembrando que os atributos salvos e mmapeados são apenas de tipos públicos. Evite nessa fase, uso de atributos complexos.
            SAMPLETABLE.INSERT(new SampleObject { Information = "Some Information", Date = DateTime.Now, Flag = true, Number = 123, Byte = 0 });
            SAMPLETABLE.INSERT(new SampleObject { Information = "More Text", Date = DateTime.Now, Flag = true, Number = 456, Byte = 16 });

            //Você também pode inserir listas desses objetos, lembre-se sempre das listas estarem sem fake id, senão você terá uma excessão.
            List<SampleObject> ListSample = new List<SampleObject> {
                new SampleObject {Information="Welcome",Date =DateTime.Now, Flag = false },
                new SampleObject {Information="To",Date =DateTime.Now, Flag = false },
                new SampleObject {Information="FakeDBCSV",Date =DateTime.Now, Flag = false }
            };

            SAMPLETABLE.INSERT(ListSample);

            //Caso ver que precisa de remover ou adicionar uma nova coluna, basta atualizar o CSV do respectivo objeto, não remova o FakeID senão dará excessão, a remoção dele é possível, mas será vista mais a diante.
            //Esta é uma etapa delicada, só faça atualizões que você tenha certeza, pois pode perder dados das colunas que NÃO EXISTAM MAIS, de forma permanente.
            SAMPLETABLE.UPDATE_CSV();

            //Vamos ver quantas ocorrencias tem na nossa Tabela?
            Console.WriteLine(
                SAMPLETABLE.COUNT_OCCURRENCES()
            );

            //E se quisermos contar quais são as valores com a Flag true??
            SAMPLETABLE.COUNT_OCCURRENCES(x => x.Flag == true);

            //Ahhh, então ele aceita expressões lambdas, interessante, o que temos de dados no nosso SELECT?
            SAMPLETABLE.SELECT().ForEach(
                x => Console.WriteLine(x.FakeId + " - " + x.Information + " - " + x.Flag + " - " + x.Number + " - " + x.Date));

            //Bom, vamos ver quais são os dados dos FakeId maiores que 3?
            SAMPLETABLE.SELECT(x => x.FakeId > 3).ForEach(
                x => Console.WriteLine(x.FakeId + " - " + x.Information + " - " + x.Flag + " - " + x.Number + " - " + x.Date));

            //Vamos atualizar os objetos Menores que 3 não tenham Byte, recebam 2
            List<SampleObject> Exemplo = SAMPLETABLE.SELECT(x => x.FakeId < 3 && x.Byte == 0);
            Exemplo.ForEach(x => x.Byte = 2);
            SAMPLETABLE.UPDATE(Exemplo);

            //Parece estar indo muito bem, vamos deletar onde tem o Byte Maior que 2
            SAMPLETABLE.DELETE(x => x.Byte > 2);

            //Nosso banco de dados Fake pode ser renomado.
            FakeDB.RENAMEDB("TESTE", "ADVANCED_TESTE");

            //Mas quais tabelas temos nesse Banco mesmo?
            List<string> NomesTabelas = FakeDB.GIVE_ME_TABLES_NAME_FROM("ADVANCED_TESTE");
            NomesTabelas.ForEach(Nomes => Console.WriteLine(Nomes));

            //Vamos ver os atributos de nossa tabela atual:
            List<string> Atributos = FakeDB.GIVE_ME_ATTRIBUTES_NAME_FROM_TABLE("SampleObject");
            Atributos.ForEach(_ => Console.WriteLine(_));

            //Vamos copiar nossa tabela, lá na NextBD que criamos lá no inicio.           
            FakeDB.EXPORT_TABLES_FOR(Path.Combine(FakeDB.pathDefault, "NextBD"), false, "SampleObject");

            //Vamos ver se a tabela chegou direito
            FakeDB.GIVE_ME_TABLES_NAME_FROM("NextBD").ForEach(x => Console.WriteLine(x));

            //Será que os dados estão lá? vamos chavear nosso SampleTable para poder ver isso para nós.
            SAMPLETABLE.SWITCH_TO_DB("NextBD");
            SAMPLETABLE.SELECT().ForEach(x => Console.WriteLine(x.FakeId + " - " + x.Information + " - " + x.Flag + " - " + x.Number + " - " + x.Date));

            //Parece que deu certo, agora, vamos dropar a tabela do nosso outro BD, o ADVANCED_TESTE.
            SAMPLETABLE.SWITCH_TO_DB("ADVANCED_TESTE");
            SAMPLETABLE.DROP_ME();
            SAMPLETABLE.SWITCH_TO_DB("NextBD");

            //Atualizaremos a information do FakeId = 1 dessa tabela
            SAMPLETABLE.UPDATE(new SampleObject { Information = "You Can More", FakeId = 1 });

            //Vamos dropar o BD anterior
            FakeDB.DROPDB("ADVANCED_TESTE");

            //Voce pode exportar as tabelas para um outro Local de forma individual, escolhendo quais tabelas quer exportar
            //Caso queira remover o Fake Id, basta inserir true no segundo parametro.
            FakeDB.CREATEDB("CONGRATULATIONS");
            FakeDB.EXPORT_TABLES_FOR(Path.Combine(FakeDB.pathDefault, "CONGRATULATIONS"), true, "SampleObject");

            //Caso queira salvar TODAS suas pastas e tabelas pode usar o BACKUP FOR
            FakeDB.BACKUP_FOR(Environment.GetFolderPath(Environment.SpecialFolder.Desktop));

            //Para eliminar totalmente a Colmeia basta dar o comando Destroy.
            FakeDB.DESTROY();
