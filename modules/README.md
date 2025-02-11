[Página principal](https://github.com/jcradavelli/lfc/blob/main/README.md)

## Submódulos
Os submódulos são conjuntos de comandos que podem ser adicionados ao programa console.
O usuário pode adicionar seus proprios submódulos nessa pasta.
Cada submódulo é composto por dois arquivos: um header público e um arquivo de código privado.

### Header público
O Header público fornece acesso à struct de configuração, que contém uma lista de tipos de funções a ser utilizado com os comandos e um ponteiro para o contexto do objeto.

``` C
typedef struct lfc_SUBMODULE_init_{
  struct {
    void (*foo) (void* handler, int arg);
    int (*foo2) (void* handler, double arg);
    void (*foo3) (void* handler);
  } functions;

  void* handler; //<! Contexto do objeto
}lfc_SUBMODULE_init_t;
```

Além da struct de configuração, expõe-se o método de registro do comando.

``` C
int lfc_register_SUBMODULE (lfc_SUBMODULE_init_t* SUBMODULE);
```


### Arquivo privado
O arquivo privado é responsável pela inicialização e registro do submódulo através da função de registro, a função de registro é responsável por adicionar cada comando de terminal.

``` C
int lfc_register_SUBMODULE (lfc_SUBMODULE_init_t* SUBMODULE)
{
  // Verifica se os ponteiros estão válidos
  lfc_SUBMODULE_init_assert(SUBMODULE);

  // Registra as funções
  cmd_foo_register(SUBMODULE)
  cmd_foo2_register(SUBMODULE);
  cmd_foo3_register(SUBMODULE);

  return(0); // comando registrado com sucesso
}
```

Cada comando é cadastrado, utilizando duas funções (register e parser) e uma estrutrura de dados (argList).

``` C
struct {
    struct arg_int *my_arg;
    struct arg_end *end;
} cmd_arglist;

int cmd_foo_parser (lfc_SUBMODULE_init_t* SUBMODULE, int argc, char **argv)
{
  lfc_SUBMODULE_init_assert(SUBMODULE);

  // Verifica se tem erro de parser
  int errors = arg_parse(argc, argv, (void **) &setNormal);
  if (errors != 0) {
      arg_print_errors(stderr, setNormal.end, argv[0]);
      return 1;
  }

  // Verifica os argumentos e chama a função adequada
  // Um comando pode ter diversas funções associadas, basta identificar a função adequada a partir dos parametros enviados ou omitidos e seus valores
  if(cmd_arglist.my_arg->count != 1)
    SUBMODULE.foo(cmd_arglist.my_arg->ival);

  return(0)// Sucess
}

void cmd_foo_register (lfc_SUBMODULE_init_t* SUBMODULE)
{
  cmd_arglist.my_arg = arg_int0("myarg", "x", "<ival>", "parametro de foo");
  cmd_arglist.end = arg_end(2);
  
  const esp_console_cmd_t cmd = {
    .command = "CMD_STRING",
    .help = "Texto de auxilio ao comando",
    .hint = NULL,
    .func_w_context = &cmd_foo_parser,
    .context = SUBMODULE,
    .argtable = &cmd_arglist,
  
  };
  esp_console_cmd_register(&cmd); //<! Essa função refistra o comando no componente de console
}
```
Os argumentos são passados pelo terminal para a função parser pela biblioteca Argtable3.
* Mais detalhes em:
  - [dine.net](https://linux.die.net/man/3/argtable)
  - [argtable3 repo](https://github.com/argtable/argtable3)
