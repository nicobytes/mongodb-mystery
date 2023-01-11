## 1. Ver las collecciones

![image](https://mystery.knightlab.com/schema.png)


```bash
show collections
```


### 2. Encontrar reporte

```js
db.crime_scene_report.find({
  date: 20180115,
  type: 'murder',
  city: 'Mongo City'
}, {
  description: 1
})
```

> Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

> Las imágenes de seguridad muestran que había 2 testigos. El primer testigo vive en la última casa de "Northwestern Dr". El segundo testigo, llamado Annabel, vive en algún lugar de "Franklin Ave".

### 3. Encontrar reporte del 1er testigo

```js
db.person.find({
  address_street_name: 'Northwestern Dr'
})
.sort({
  address_number: -1
})
```

> 14887 => Morty Schapiro

```js
db.interviews.find({person_id: 14887})
```

> I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

> Escuché un disparo y luego vi a un hombre salir corriendo. Tenía una bolsa de "Get Fit Now Gym". El número de membresía en la bolsa comenzaba con "48Z". Solo los miembros dorados tienen esas bolsas. El hombre subió a un automóvil con una placa que incluía "H42W".

```js
db.people.aggregate([
  {
    $match: {
      address_street_name: 'Northwestern Dr'
    },
  },
  {
    $lookup: {
      from: 'interviews',
      localField: 'id',
      foreignField: 'person_id',
      as: 'interview'
    },
  },
  {
    $sort: {
      address_number: -1
    },
  },
  {
    $limit: 1
  }
])
```

### 3. Encontrar reporte del 2do testigo

```js
db.people.find({
  address_street_name: 'Franklin Ave',
  name: { $regex: /Annabel/, }
})
```

> 16371 => Annabel Miller

```js
db.interviews.find({person_id: 16371})
```

> I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

> Vi el asesinato y reconocí al asesino de mi gimnasio cuando estaba entrenando la semana pasada, el 9 de enero.

```js
db.people.aggregate([
  {
    $match: {
      address_street_name: 'Franklin Ave',
      name: { $regex: /Annabel/, }
    },
  },
  {
    $lookup: {
      from: 'interviews',
      localField: 'id',
      foreignField: 'person_id',
      as: 'interview'
    },
  },
  {
    $limit: 1
  }
])
```


### 4. Analizar datos

- Morty Schapiro

- Gym: Get Fit Now Gym
- Membership number: 48Z...
- Member: Gold
- Plate: H42W...

- Annabel Miller

- Date: 9 de enero

### 5. Encontrarlo

```js
db.get_fit_now_member.aggregate([
  {
    $match: {
      membership_status: 'gold',
      id: { $regex: /^48Z/, }
    },
  },
  {
    $lookup: {
      from: 'get_fit_now_check_in',
      localField: 'id',
      foreignField: 'membership_id',
      as: 'check_in',
    },
  },
  {
    $match: {
      'check_in.check_in_date': 20180109,
    },
  },
])
```

```js
db.get_fit_now_member.aggregate([
  {
    $match: {
      membership_status: 'gold',
      id: { $regex: /^48Z/, }
    },
  },
  {
    $lookup: {
      from: 'get_fit_now_check_in',
      localField: 'id',
      foreignField: 'membership_id',
      as: 'check_in',
    },
     $lookup: {
      from: 'people',
      localField: 'id',
      foreignField: 'person_id',
      as: 'person',
    },
  },
  {
    $match: {
      'check_in.check_in_date': 20180109,
    },
  },
])
```

> 67318 => Jeremy Bowers

### 6. Pero...

```js
db.people.aggregate([
  {
    $match: {
      id: 67318,
    },
  },
  {
    $lookup: {
      from: 'interviews',
      localField: 'id',
      foreignField: 'person_id',
      as: 'interview'
    },
  },
])
```

> I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5 (65) or 5'7 (67). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

> Me contrató una mujer con mucho dinero. No sé su nombre, pero sé que mide alrededor de 5'5 (65) o 5'7 (67). Es pelirroja y conduce un Tesla Model S. Sé que asistió al Concierto Sinfónico de SQL 3 veces en diciembre de 2017.
