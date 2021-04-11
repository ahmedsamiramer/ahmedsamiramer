- 👋 Hi, I’m @ahmedsamiramer
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
ahmedsamiramer/ahmedsamiramer is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->var getRobberies = function(done){
	console.log('%c Soygun seçiliyor.','background: black; color: white')
	$.ajax({
  		type: "GET",
  		url: 'https://www.thecrims.com/api/v1/robberies',
  		success: function(res){
  			done(res.single_robberies
                           .filter(robbery => robbery.successprobability == 100)
                           .sort((a, b) => b.difficulty - a.difficulty)
                           .find((x, index) => index == 0))
  		}
	});
}

var getNightClubs = function(done){
	console.log('%c Klüplere bakılıyor.','background: black; color: white')
	$.ajax({
  		type: "GET",
  		url: 'https://www.thecrims.com/api/v1/nightclubs',
  		success: function(res){
  			done(res.favorites.length > 0 ?  res.favorites : res.nightclubs)
  		}
	});
}

var joinClub = function(club,done){
	console.log('%c '+club.name + ' klübüne girdi.','background: black; color: white')
	$.ajax({
  		type: "POST",
  		url: 'https://www.thecrims.com/api/v1/nightclub',
  		data: {id:club.id,mm:[],mc:[],kp:[]},
  		success: function(res){
  			done(res)
  		},
  		error: function(){
  			console.log('%c Bilet bitti :(','background: red; color: white')
  		}
	});
}

var drink = function(drug,done,currentStamina){	
	var amount = currentStamina ? parseInt((100-currentStamina)/drug.stamina) : 1;
	if(amount > 99){
		amount = 99
	}
	console.log('%c ' +amount +' adet ' + drug.name + ' içiliyor','background: yellow; color: black')
	
	$.ajax({
  		type: "POST",
  		url: 'https://www.thecrims.com/api/v1/nightclub/drug',
  		data: {id: drug.id, quantity: amount, mm: [], mc: [], kp: []},
  		success: function(res){
  			console.log('%c Güç: ' + res.user.stamina,'background: green; color: white')
  			if(res.user.stamina < 100){
  				drink(drug,done,res.user.stamina)
  			}else{
  				if(res.user.addiction > 5){
  					heal(res.user.addiction,done);
  				}else{
  					done()
  				}
  			}
  		},
  		error: function(){
  			power(done)
  		}
	});
}

var heal = function(quantity,done){
	console.log('%c Tedaviye gidiliyor.','background: black; color: white')
	$.ajax({
  		type: "POST",
  		url: 'https://www.thecrims.com/api/v1/hospital/healing',
  		data: {id:10,quantity:quantity,mm:[],mc:[],kp:[]},
  		success: function(res){
  			console.log('%c Bağımlılık: ' + res.user.addiction, 'background: green; color: white')
  			if(res.user.addiction > 0){
  				heal(res.user.addiction,done)
  			}else{
  				done()
  			}
  		}
	});
}

var power = function(done){
	getNightClubs(function(clubs){
		var clubIndex = Math.floor(Math.random() * Math.floor(clubs.length));
		joinClub(clubs[clubIndex],function(res){
			var drugIndex = Math.floor(Math.random() * Math.floor(res.nightclub.products.drugs.length));
			drink(res.nightclub.products.drugs[drugIndex],done)
		})
	})
}

setInterval(function() { 
  if(+document.querySelector('#user-profile-stamina').innerText.replace(/[^0-9]/g,'') < 25) {
    power(()=>{}); 
    document.querySelector('#user-profile-stamina').innerText="100%"
  } 
}, 2500);
