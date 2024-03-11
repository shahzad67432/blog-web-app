`
### ROutes Backup
//app.use('/api/v1/blog/*', async(c, next)=>{
  const header = c.req.header("Authorization") || " ";
  const token = header.split(" ")[1]
  const response = await verify(token, c.env.JWT_SECRET)
  if(response.id){
    next()
  }else{
    c.status(403)
    return c.json({
      error: "authentication faled out",
    })
  }
})

app.post('/api/v1/signup', async (c) => {
	const prisma = new PrismaClient({
		datasourceUrl: c.env?.DATABASE_URL	,
	}).$extends(withAccelerate());

	const body = await c.req.json();
	try {
		const user = await prisma.user.create({
			data: {
				email: body.email,
				password: body.password,
        		firstname: body.firstname,
        		lastname: body.lastname
			}
		});
		const jwt = await sign({ id: user.id }, c.env.JWT_SECRET);
		return c.json({ jwt });
	} catch(e) {
		c.status(403);
		return c.json({ error: "error while signing up" });
	}
})


app.post('/api/v1/signin', async (c) => {
	const prisma = new PrismaClient({
		datasourceUrl: c.env?.DATABASE_URL	,
	}).$extends(withAccelerate());

	const body = await c.req.json();
	const user = await prisma.user.findUnique({
		where: {
			email: body.email
		}
	});

	if (!user) {
		c.status(403);
		return c.json({ error: "user not found" });
	}

	const jwt = await sign({ id: user.id }, c.env.JWT_SECRET);
	return c.json({ jwt });
})


app.post('/api/v1/blog', async (c) => {
	const userId = c.get('userId');
	const prisma = new PrismaClient({
		datasourceUrl: c.env?.DATABASE_URL	,
	}).$extends(withAccelerate());

	const body = await c.req.json();
	const post = await prisma.post.create({
		data: {
			title: body.title,
			content: body.content,
			authorId: userId,
			author: body.author,
		}
	});
	return c.json({
		id: post.id
	});
})

app.put('/api/v1/blog', async(c) =>{
	const prisma = new PrismaClient({
		datasourceUrl: c.env?.DATABASE_URL	,
	}).$extends(withAccelerate());

	const body = await c.req.json();
	const post = await prisma.post.update({
		where: {
            id: body.id
        },
        data: {
            title: body.title,
            content: body.content
        }
	})
	return c.json({
        msg:"post updated successfully"
    });
})
app.get('/api/v1/blog/:id', async(c) => {
  const id = c.req.param('id')
  const prisma = new PrismaClient({
    datasourceUrl: c.env?.DATABASE_URL    ,
  }).$extends(withAccelerate());

  const post = await prisma.post.findUnique({
	    where: {
            id: id
        }
  })
  return c.json(post)
})
app.get('/api/v1/blog/bulk', (c) => {
  const prisma = new PrismaClient({
    datasourceUrl: c.env?.DATABASE_URL  ,
  }).$extends(withAccelerate());

  const posts = prisma.post.findMany({
    select:{
      title: true,
      content: true,
      author: true,
    }
  })
  return c.json(posts)
})
`