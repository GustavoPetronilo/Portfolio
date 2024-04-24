# Sobre o projeto

Desenvolvido em Jupyter Notebook, deixá-lo viável na forma de aplicação necessitou do
desenvolvimento em uma framework, a escolhida foi o Django devido a linguagem utilizada
que é o Python. Existe a framework para Python chamada de Flask, há diferenças, nada
impede de passar o desenvolvimento de Django para Flask, mas há modificações que são
necessárias.


## Framework Django

A framework do Django é utilizada para desenvolvimento de aplicações em Python e com a
possibilidade de disponibilizar as aplicações desenvolvidas em servidores da web.

## Código (Funções)

### def index(request)
Função que carrega a tela inicial chamada ‘welcome.html’.

```python 
def index(request):
    return render(request, template_name: 'welcome.html')
```

### def welcome(request)
Função responsável em fazer requisições, sendo elas ‘manual’ que carregará a página
‘create_point.html’, responsável por inserir dados manuais e ‘upload’ que carregará a página
‘upload.html’ que permitirá inserir arquivo com informações.
```python 
def welcome(request):
    if 'manual' in request.POST:
        return render(request, 'create_point.html')
    if 'upload' in request.POST:
        return render(request, 'upload.html')
```

### def upload(request)
Função que carrega a tela ‘upload.html’.

```python
def upload(request):
    global myfile, filename, full_path, uploaded_file
    if 'help_upload' in request.POST:
        return render(request, 'help_upload.html')
    myfile = request.FILES['myfile']
    fs = FileSystemStorage()
    filename = fs.save(myfile.name, myfile)
    uploaded_file = fs.url(filename)
    print(uploaded_file)
    s3 = boto3.client('s3')
    my_bucket = 'elasticbeanstalk-us-west-2-811991846412'
    response = s3.upload_file(Filename=filename, Bucket=my_bucket, Key=myfile.name)
    folder = s3.upload_file(
        Filename=filename,
        Bucket=my_bucket,
        Key=filename
    )
    N, pontos = input_upload(folder, filename)

    pontos_osm, pontos_maps, enderecos, cidades = proc_enderecos(request, N, pontos)
    context = {}
    n_times = 1
    distancia = 0
    new_points = []
    melhor_rota = []

    import time

    # start_time = time
    nome = 'time_process' + str(time.time())
    arq = open(nome, 'a')
    back_start = False
    for i in range(n_times):
        start_time = time.time()
        melhor_r, melhor_s_p, new_p, dist, G = QTSP(request, N, pontos_osm, cidades, back_start)
        finish_time = time.time()
        arq.write('Time Process: ' + str(start_time) + ' to ' + str(finish_time) + ': ' + str(finish_time - start_time))
        if distancia == 0 or dist < distancia:
            melhor_rota = melhor_r
            new_points = new_p
            distancia = dist
            melhor_seq_points = melhor_s_p
    arq.close()

    print('Distância: ' + str(distancia))
    print('Melhor_rota: ' + str(melhor_rota))
    print('Sequência de Pontos: ' + str(new_points))

    disc_seq = []
    pontos_disc = []
    for i in range(len(new_points)):
        for j in range(len(melhor_seq_points)):
            if int(new_points[i]) == int(melhor_seq_points[j]):
                disc_seq.append(j)
    for i in melhor_seq_points:
        if i in new_points:
            pontos_disc.append(i)
    print('Pontos discriminados: ' + str(pontos_disc))

    print('Sequência discriminada: ' + str(disc_seq))

    rota_aux = []
    direcionais = []
    start = melhor_rota[0]
    finish = melhor_rota[len(melhor_rota) - 1]

    waypoints = []
    for k in range(len(melhor_rota)):
        rota_aux.append(melhor_rota[k])
        if k % 25 == 0:
            direcionais_aux, key, start_aux, finish_aux, waypoints_aux = Directions(request, N, rota_aux)
            direcionais.append(direcionais_aux)
            waypoints.append(waypoints_aux)
            rota_aux = []
            rota_aux.append(melhor_rota[k])
        if k % 25 != 0 and k == len(melhor_rota) - 1:
            direcionais_aux, key, start_aux, finish_aux, waypoints_aux = Directions(request, N, rota_aux)
            direcionais.append(direcionais_aux)
            waypoints.append(waypoints_aux)

    pontos = []
    pontos1 = []
    chaves = sorted(pontos_maps.keys())
    for i in chaves:
        auxiliar1 = [pontos_maps[i][0], pontos_maps[i][1]]
        pontos1.append(auxiliar1)

    for t in pontos_disc:
        auxiliar = [G.nodes[t]['y'], G.nodes[t]['x']]
        pontos.append(auxiliar)

    start = pontos[0]
    finish = pontos[len(pontos) - 1]
    context['direcionais'] = direcionais
    context['start'] = start
    context['finish'] = finish
    context['pontos'] = pontos

    return render(request, 'map.html', context)
```



```python

```
